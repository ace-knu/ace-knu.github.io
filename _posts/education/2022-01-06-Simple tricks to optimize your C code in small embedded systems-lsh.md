---
layout: single
title: "Simple tricks to optimize your C code in small embedded systems"
categories:
    - education
tags: 
    - C/C++
    - Linux
    - Embedded
---

## 최적화란?
- 최적화란 주어진 조건이나 범위 내에서 최대의 효율을 발휘하게 만드는 것을 의미
- 임베디드 소프트웨어는 주로 메모리 공간이 제한된 마이크로컨트롤러에서 실행되므로 코드 최적화가 필수적
- 실시간 및 임베디드 시스템용으로는 어셈블리어가 가장 좋지만, 거의 대부분의 하드웨어 엔지니어는 C언어를 사용하는 것을 선호
- 컴파일러가 더 좋은 코드를 만들어 낼 수 있도록 코드 작성 시 C언어 문법을 구성하는 것이 중요


## 1. 수학 표현식의 적절한 사용

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{<br> &nbsp;int a, b;<br> &nbsp;a = (b - 1) * 3;<br> }|void main(void)<br>{<br>&nbsp;int a, b ;<br>&nbsp;a = (b – 1) ;<br>&nbsp;a = a + a + a ;<br>}|
|Program memory bytes used = 50|Program memory bytes used = 25|

- 덧셈(뺼셈) > 곱셈 > 나눗셈 순으로 더 빠른 연산이 가능하므로 이를 고려하여 프로그래밍 하는 것이 중요


|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{<br>&nbsp;float a, b, c ;<br> &nbsp;a = (b + c) * 0.1 ;<br>}|void main(void)<br>{<br>&nbsp;float a, b, c ;<br>&nbsp;a = (b + c) / 10.0 ;<br>}|
|Program memory bytes used = 570|Program memory bytes used = 559|

- 연산식에 최대한 정수로만 구성될 수 있도록 하는 것이 메모리 사용량이 더 좋음


## 2. integer type을 사용
- 정수 수준에서 산술 연산과 매개변수 전달을 수행하므로, int를 사용하여 숫자를 유지하는 것이 좋음
- char를 사용하는 경우 컴파일러는 먼저 값을 정수로 변환하고 연산을 수행한 후 결과를 다시 char형으로 반환함
    - 이를 Integer Promotion(정수 승격)이라고 함

```c
char sum_char(char a, char b)
{
    char c;
    c = a + b;
    return c;
}
 
int sum_int(int a, int b)
{
    int c;
    c = a + b;
    return c;
}  
```
- 위 코드의 순서는 다음과 같다.
    - **sum_char:**
        1. sign extension을 통해 두 번째 parameter를 int로 변환
        2. b와 같이 스택에 sign extended parameter를 push
        3. 첫 번째 parameter도 1, 2의 과정을 수행
        4. 호출된 함수는 a와 b를 더함
        5. 결과값은 char로 cast되고, char c에 저장됨
        6. c는 다시 sign extension을 수행하고, sign extended c는 return register에 복사되고, 호출자에게 반환됨
        7. 호출자는 다시 int에서 char로 변환함
        8. 결과가 저장됨
    - <b>sum_int:</b>
        1. 스택에 a push
        2. 스택에 b push
        3. 호출된 함수는 a와 b를 더함
        4. 결과는 int c에 저장됨
        5. c은 return register에 복사되고, 함수는 호출자에게 반환
        6. 호출된 함수는 반환값을 저장

- 따라서 스토리지 요구 사항으로 인해 char 또는 short를 사용해야만 하는 경우 외에는 int 사용하는 것이 좋음
- 만약 char 및 short를 사용해야 하는 경우, Byte alignment and ordering의 영향을 고려해 공간을 절약하여 사용할 수 있도록 해야함


## 3. 변수의 적절한 활용
### 3.1 지역 변수 최소화
- 지역 변수의 개수가 적어질수록 컴파일러는 레지스터에 변수들을 적절하게 할당 가능
    - 스택에 저장된 지역변수보다 레지스터에 직접 접근하는 것이 더욱 빠르므로 효율 증가
- 스택에 보관된 지역 변수에 대한 프레임 포인터 작업을 피할 것
- 모든 지역 변수는 레지스터에 있으므로 메모리에서 액세스하는 것보다 성능이 향상됨
    - 스택에 지역 변수를 저장할 필요가 없는 경우, 컴파일러는 프레임 포인터를 설정하고 복원하는 오버헤드를 발생시키지 않음

### 3.2 매개 변수의 수 최소화 
- 매개변수가 많은 함수의 호출은 각 호출에서 스택에 대해 더 많은 매개변수의 push - pop이 필요함
    - 이것은 정수, 포인터 등과 같은 일반 유형에 적합하며, 일반적으로 4바이트로 제한
    - data type이 커질수록, 스택에 있는 개체를 복사하는 비용이 커짐
        - 스택에 생성되는 복사본에 대한 생성자를 호출하는 추가 오버헤드가 있으며, 함수가 종료되면 소멸자도 다시 호출됨
-  따라서 참조를 매개변수로 전달하는 것이 효율적
    - 임시 개체 생성, 복사 및 파괴의 오버헤드를 줄일 수 있음
    - 이 최적화는 pass by value 매개변수를 const 참조로 대체하여 코드에 큰 영향을 미치지 않고 쉽게 수행할 수 있음

### 3.3 배열의 차원 최소화

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:----------|
|void main(void)<br>{<br><br> &nbsp;const char *TabMois[] = { “Jan”, ”Feb”, ”Mar”, ”Apr”, ”May”, ”Jun”, “Jul”, ”Aug”, ”Sep”, ”Oct”, ”Nov”, ”Dec” };<br><br> &nbsp;char ch1 = TabMois[0][0], ch2 = TabMois[0][1], ch3 = TabMois[0][2] ;<br><br>}|void main(void)<br>{<br><br>&nbsp;const char TabMois[] = “JanFebMarAprMayJunJulAugSepOctNovDec” ;<br><br> &nbsp;char ch1 = TabMois[0], ch2 = TabMois[1], ch3 = TabMois[2] ;<br><br>}|
|Program memory bytes used = 149|Program memory bytes used = 68|

- 단순히 인덱싱 체계의 차이 뿐, 특정 인덱스에 액세스하는 속도는 동일
    - 컴파일러가 1차원 배열로 매핑을 수행   
    - 인덱스 표현식은 포인터 산술 표현식으로 컴파일되므로 차이 없음
- 하지만, 루프를 어떻게 도느냐에 따른 성능차가 있을 수도 있음
- 2차원 배열에서 행단위가 아닌 열단위로 배열에 액세스할 경우 오류 발생 가능
    - 임베디드 프로그래밍 시 주로 사용하는 언어에서는 메모리에서 다차원 배열 저장 시 행 우선 저장을 하기 때문


### 3.4 `unsigned int`를 사용

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{<br><br>&nbsp;int Vbat ; // 1, 2, …, 1023<br><br> &nbsp;Vbat = 1023 / Vbat ;  // case 1<br> &nbsp;Vbat = (2048L * 15) / Vbat ;  // case 2<br><br>}|void main(void)<br>{<br><br>&nbsp;int Vbat ; // 1, 2, …, 1023<br><br> &nbsp;Vbat = 1023U / Vbat ;  // case 1<br> &nbsp;Vbat = (2048UL * 15) / Vbat ;  // case 2<br><br>}|
|Case 1 : Program memory bytes used = 102<br>Case 2 : Program memory bytes used = 160|Case 1 : Program memory bytes used = 73<br>Case 2 : Program memory bytes used = 114|

- unsigned int가 더 효율적이고 항상 더 빠른 코드를 생성함
- signed int에 대한 명령어 셋의 지원이 부족하기 때문에 signed integer를 사용하면 컴파일러가 라이브러리나 함수를 사용해 작업을 수행해야함
    - 이에 비해서 unsigned int는 명령어셋이 완전히 정의됨
    - shift연산이 2<sup>N</sup>으로 곱하거나 나누는 연산과 동일함
    - 더욱 효율적이며 안전한 연산 가능
- signed int의 overflow발생 시 
- 레지스터값은 거의 대부분 unsigned entities로 취급되며, 임베디드 시스템은 레지스터값을 처리하는데 많은 시간을 소비하므로 unsigned형이 더 효율적


## 4. 적절한 조건문 사용
- 프로그램 실행 시간은 선택된 조건문의 구조에 크게 의존적일 수 있음
- 확인해야 할 조건의 개수에 따라 적절한 구문 사용이 필요

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{<br><br>&nbsp;int a, b ;<br><br> &nbsp;if (a – b) b = (a – 5) * 100 ;    // case 1<br> &nbsp;if (b > a) b = (a – 5) * 100 ;     // case 2<br> &nbsp;if (b – a > 0) b = (a – 5) * 100 ; // case 3<br><br>}<br>|void main(void)<br>{<br><br>int a, b ;<br><br>&nbsp;if (a != b) b = (a – 5) * 100 ;    // case 1<br>&nbsp;if (a < b) b = (a - 5) * 100 ;      // case 2<br>&nbsp;if (a – b < 0) b = (a - 5) * 100 ; // case 3<br><br>}|
|Case 1 : Program memory bytes used = 67<br>Case 2 : Program memory bytes used = 62<br>Case 3 : Program memory bytes used = 77|Case 1 : Program memory bytes used = 58<br>Case 2 : Program memory bytes used = 62<br>Case 3 : Program memory bytes used = 65|

- 더 자주 발생하는 케이스를 우선 배치하면, 자주 발생하는 케이스에 대해 비교 연산 횟수를 줄일 수 있음
    - 첫 번째 케이스만 실행한 후, 다음 코드로 계속 이동하기 때문
    - 클록 주기가 절약되고 코드가 더 명확해짐


### 4.1 확인해야 할 조건이 적은 경우

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{ <br><br>&nbsp;int a, b;<br><br>&nbsp;switch(a)<br>&nbsp;{<br>&nbsp;&nbsp;case 0: b = a + 5; break; <br>&nbsp;&nbsp;case 1: b = a * 5; break; <br>&nbsp;&nbsp;case 2: b = a - 5; break;<br>&nbsp;}<br><br> }|void main(void)<br>{<br><br>&nbsp;int a, b ;<br><br> &nbsp;if(a == 0) b = a + 5;<br> &nbsp;if(a == 1) b = a * 5;<br>&nbsp;if( a==2 ) b = a - 5;<br><br>}|
|Program memory bytes used = 79|Program memory bytes used = 74|

- switch문 대신 if문을 이용하면 메모리 사용량을 줄일 수 있음

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{ <br><br> &nbsp;int a, b, c;<br><br>&nbsp;if(a == 0)<br>&nbsp;&nbsp;b = 0; <br><br>&nbsp;else<br>&nbsp;&nbsp;b = (a - c) * 100;<br><br>}|void main(void)<br>{<br><br>&nbsp;int a, b, c ;<br><br> &nbsp;b = 0;<br><br> &nbsp;if( a )<br>&nbsp;&nbsp;b = (a - c) * 100;<br><br>}|
|Program memory bytes used = 64|Program memory bytes used = 63|

- 메모리 사용량의 차이가 무시해도 될 만큼 크지는 않지만, 위 같은 코드가 반복적으로 사용된다면 이를 이용해 덜 빈번하게 조건문을 실행할 수 있음

![jb20130305listing5](https://www.edn.com/wp-content/uploads/media-1179598-jb20130305listing5.png)
- if만 사용하는 경우보다 else 또한 적절하게 사용하는 경우, 첫 번째 케이스만 평가된 후 다음 코드로 계속 이동하므로 클록 주기가 절약되고 코드가 더 명확해짐


### 4.2 확인해야 할 조건이 많은 경우
- 컴파일러의 특성이나 코드의 상황에 따라 더 효율적인 조건문이 달라짐
    - if-else문이 switch문보다 메모리 사용 효율이 더 낮을 수도 있음
- 각 케이스에 대한 레이블을 jump table로 구현하는 경우, 각 케이스 레이블이 멀리 떨어져 있는 if-else코드보다 switch문이 훨씬 빠른 실행이 가능함
    - 이때, jump table기반 switch문의 성능은 case의 개수와는 무관
    - 이 경우, 각 label은 goto와 비슷한 성격을 갖게 됨
- 위 같은 컴파일러의 특성을 고려하지 않더라도, 코드의 흐름을 이해하기에는 switch문이 더 쉽기 때문에 프로세서에서 추가 클록 주기를 짜서 더 효율적인 메모리 사용이 가능함

![5LzeXYanX94B1UKFKsQVuG44KFLjWJTtYpXrog](https://cafeptthumb-phinf.pstatic.net/MjAyMjAxMDZfMTUx/MDAxNjQxNDA5MTIzNDc3.Pe99dDXVSM5zyLCIY4trISWn-LctCFenLpRet9h5tbIg.zKZzj-5LzeXYanX94B1UKFKsQVuG44KFLjWJTtYpXrog.PNG/mk.PNG?type=w1600)


## 5. 효율적인 반복문 사용
- 컴퓨터 프로그래밍이 초기 단계에 있을 때, 프로그램의 흐름은 `goto`문으로 제어됨
    - 현재 코드를 중단하고 말 그대로 코드의 다른 섹션으로 이동할 수 있는 함수임
- 프로그래밍 언어가 함수의 개념을 통합하기 시작하면서 for문과 while문을 이용해 대체할 수 있게됨
- 따라서 goto문 보다는 for문이나 while문을 지향하는 것이 더욱 효과적
    - for문 사용 예시
![jb20130305listing3](https://www.edn.com/wp-content/uploads/media-1179596-jb20130305listing3.png)
    - while문 사용 예시
![jb20130305listing4](https://www.edn.com/wp-content/uploads/media-1179597-jb20130305listing4.png)

- for문의 조건문을 적절하게 사용하면 코드의 성능을 향상시킬 수 있음

|<b>Example: </b>|<b>Use instead: </b>|
|:---------|:------------|
|void main(void)<br>{<br><br> &nbsp;for (a = 10 ; a > 0 ; a–)<br>&nbsp;b = c + a ;<br>}|void main(void)<br><br>{<br><br> &nbsp;for (a = 10 ; a– ; ) <br>&nbsp;b = c + a ;<br><br>}|
|Program memory bytes used = 43|Program memory bytes used = 27|


## 부록
### 1. Cache 
#### 1.1 Cache란?
- 프로그램이 수행될 때 나타나는 지역성을 이용하여 메모리나 디스크에서 사용되었던 내용을 특별히 빠르게 접근할 수 있는 곳에 보관하고 관리함으로써 이 내용을 다시 필요로할 때 보다 빠르게 참조하도록 하는 것
- 즉, 사용되었던 데이터는 다시 사용되어 질 가능성이 높다는 개념을 이용해 재사용 확률이 높은 데이터를 좀 더 빠르게 접근 가능한 저장소를 사용한다는 개념


#### 1.2 Cache Hit, Cache Miss
- 캐시 히트란 CPU가 참조하고자 하는 메모리가 캐시에 존재할 경우를 의미하며, 캐시 미스란 캐시에 참조하고자 하는 메모리가 존재하지 않을때를 의미함
- 캐시와 메인 메모리 사이에 데이터가 오가는 것 역시 많은 시간을 소요하므로 캐시에 한 번에 일정량의 주소를 한 번에 매핑해두고 다른 주소에 의해 교체될 때까지 사용
- 공간 집약성과 코드 집약성을 높일수록 더 빠른 퍼포먼스를 보여줌

### 2. [Division vs multiplication](http://highperformanceinjava.blogspot.com/2013/02/division-vs-multiplication.html)

### 3. [Integer Promotions-1](https://www.geeksforgeeks.org/integer-promotions-in-c/) <br> [Integer Promotions-2](https://wiki.sei.cmu.edu/confluence/display/c/INT02-C.+Understand+integer+conversion+rules)

### 4. [Integer vs Unsigned integer1](https://embeddedgurus.com/stack-overflow/tag/unsigned/)<br>
[Integer vs Unsigned integer2](https://embeddedgurus.com/stack-overflow/2009/07/efficient-c-tips-10-use-unsigned-integers/)

### 5. [Duff Device](https://en.wikipedia.org/wiki/Duff%27s_device)


### 6. Reference
- <https://www.embedded.com/10-simple-tricks-to-optimize-your-c-code-in-small-embedded-systems/>
- <https://www.edn.com/10-c-language-tips-for-hardware-engineers/>
- <https://www.eventhelix.com/embedded/optimizing-c-and-cpp-code/>
- <http://highperformanceinjava.blogspot.com/2013/02/division-vs-multiplication.html>
- <https://embeddedgurus.com/stack-overflow/2009/07/efficient-c-tips-10-use-unsigned-integers/>