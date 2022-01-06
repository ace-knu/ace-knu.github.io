---
layout: single
title: "Manipulating C strings safely"
categories:
    - education
tags: 
    - C/C++
    - Linux
---

## <string.h>
- 문자열을 복사, 비교, 연결, 부분을 선택하는 등 문자열 연산 처리를 위한 여러 가지 함수와 매크로 형식을 제공하고 있음
- '\0'(혹은 NULL문자)를 기준으로 문자열을 구분하며 NULL문자 뒤 공백이나 다른문자가 존재하더라도 출력 시에는 NULL문자까지 읽음
- 이러한 특징을 활용하여 문자열의 연산을 진행하지만,시스템에서는 중대한 오류를 발생시키기도 함




## 발생 가능한 오류
- 헤더파일에 정의된 `strcpy()`나 `strcat()`와 같은 특정 함수를 통해 `\0`을 기준으로 문자열을 복사하거나 병합할 때 오류가 발생됨
- 하나 이상의 인수가 NULL문자로 적절하게 종료되지 않을 경우, memory overwrite와 같은 문제를 일으킴 
- 일반적으로 발생한 문제가 즉시 나타나지 않으므로 고장의 원인을 찾기 어려움

### 오류 발생 예 
```c
char frame[12], *str;   
…  
str = Geo_getLatitude(position);
frame = strcat(frame, str);
```
- frame과 str이 가리키는 메모리에 저장된 문자열은 아래와 같음

![f1](https://www.embedded.com/wp-content/uploads/2021/12/12lf_f1.png)

![f2](https://www.embedded.com/wp-content/uploads/2021/12/12lf_f2.png)
- 두 문자열에 대해 strcat()를 수행하면 아래와 같은 결과를 출력함

![f3](https://www.embedded.com/wp-content/uploads/2021/12/12lf_f3.png)




### 문제점
1. frame에 할당된 저장공간이 실제 결과보다 부족하여 하위 3바이트에 대해 memory overwriting 발생
    - memory overwriting으로 인해 메모리 내에 기존에 저장된 값이 손상됐을 가능성 있음
    - 결과값 출력 시에는 정상적으로 동작하므로 오류를 찾아내기 어려움
2. 최종 frame의 문자열의 종단부가 NULL문자가 아니므로 다른 문자열 연산 시 문제가 발생할 가능성이 있음 




### 해결책
1. 소스 문자열 str의 크기가 예상한 길이보다 큰지 확인하는 함수 생성
    - 예상한 길이보다 크면 false를 반환
    - false가 반환되면 str이 삭제되고 빈 문자열이 대신 사용됨

2. 문자열 내 '\0'이 있는지 확인하는 함수 생성
    - NULL문자가 존재하면 포인터에 해당 문자열을 가리키는 주소를 반환하고, 반대의 경우는 NULL값 리턴


#### 함수 구현

```c
static const char empty[] = "????";
...
res = SecString_strchk(str, MAX_LENGTHOF_STR);
frame = strcat(frame, (res == false) ? empty: str));
```
- `SecString_strchk()`의 연산 결과를 res에 저장
    - res 값이 false면 빈 문자열을 반환함
    - res 값이 true면 입력된 문자열의 주소값을 반환함


```c
bool SecString_strchk(char *s, size_t num)
{
    char * pos = memchr(s, '\0', num);
    return (pos == (char *)0) ? false: true;
}
```

- source 문자열의 예상한 길이범위 내에서 '\0'이 있는지 확인
- `SecString_strchk()`는 문자열 str의 길이가 `MAX_LENGHTOF_STR`보다 큰지를 확인함   
    - 범위 내에 `NULL`이 존재하지 않는다면, 포인터 pos에는 NULL 값 저장, false 리턴
    - 범위 내에 `NULL`이 존재한다면, 포인터 pos에는 문자열 str의 주소값이 저장되며, true 리턴

```c
size_t SecString_strnlen(char *s, size_t maxlen)
{
    char *pos = memchr(s, '\0', maxlen);
    return (pos != (char *)0) ? (pos - s): maxlen;
}
```
- `SecString_strnlen()`역시 위의 함수와 동일한 결과를 냄
    - 기존의 `strlen()`의 안전한 버전
    - 하지만 문자열이 `MAX_LENGHTOF_STR`보다 클 때에 대해서는, 이 함수를 사용했을 때 도달할 수 있는 안전성에 한계가 있음



#### 적용 예시

```c
/** \file Geo.c */
#include "SecString.h"
...
static char *getAttribute(char *attribute, size_t bufSize)
{
    char *pos = (char *)0;
    bool res;
    res = SecString_strchk(attribute, bufSize);
    if (res == false)
    {
        if (errorHandler != (GeoErrorHandler)0)
        {
            errorHandler(INDEX_OUT_OF_RANGE);
        }
    }
    else
    {
        pos = attribute;
    }
    return pos;
}


char *Geo_getLatitude(Geo *const me)
{
    return getAttribute(me->latitude, LATITUDE_LENGTH + 1);
}


#include "Geo.h"
...
static const char empty[] = "";
...
value = Geo_getLatitude(position);
frame = strcat(frame, (value == (char *)0) ? empty: value);
```

- `Geo_getLatitude()`가 호출되면 [me->latitude]의 문자열 길이를 검사하는 `getAttribute()`를 호출하여 `LATITURE_LENGH + 1`을 초과하는지에 대해 조사함
    - 문자열의 길이가 `LATITURE_LENGH + 1`을 초과하면, `errorHandler()`를 호출하고 `NULL`을 반환함
    - 문자열 길이가 `LATITURE_LENGH + 1`자를 초과하지 않으면, 포인터 변수 position이 가리키는 주소값을 반환함
- 반환된 해당 주소값을 사용하여 `strcat()`이 수행이 됨
    - source 문자열의 길이와 '\0'유무를 미리 판단하고 `strcat()`을 수행하므로, 안정적으로 동작하는 것을 알 수 있음


## 위 코드로 얻을 수 있는 이점
무결성 보장이 어려운 환경(파일, 비휘발성 메모리 장치, 통신 프로토콜 메시지 등) 에서 문자열을 조작하는 C 언어로 작성된 소프트웨어 시스템에 대하여, 문자열 조작에 무결성을 보장하기위해 이 방법을 적용할 수 있음




---



## 부록
### 사용된 string 함수
```c
char *strcpy(char *des, const char *src);
```
- des 주소가 char* 형으로 리턴됨
- src에서 '\0'을 찾아 해당 부분 까지 des로 복사함

```c
void *memchr(const void *ptr, int value, size_t mum);
```
- 검색하고자 하는 메모리 블럭의 주소를 ptr로 나타내고, 찾고자 하는 문자 value를 탐색함
- num 사이즈만큼의 문자열에서 value값을 찾아냄
- value를 찾았을 경우 value의 주소를 리턴하나 아닌 경우 NULL을 리턴함

```c
char *strcat(char *des, const char *src);
```
- des 문자열 뒤에 src 문자열이 복사됨
- '\0'이 각 문자열 마지막에 존재해야하며 des 문자열이 src문자열 + '\0'을 복사할 만큼의 메모리 공간이 필요함.

### Reference
- [Manipulating C strings safely](https://www.embedded.com/manipulating-c-strings-safely/)
