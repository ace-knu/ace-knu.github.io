---
layout: single
title: "Firmware security – preventing memory corruption and injection attacks"
categories:
    - education
tags: 
    - C/C++
    - Linux
    - Embedded
--- 
  
---  
임베디드 개발자에게는 보안문제가 크게 우선순위로 생각하는 않는 분위기이다. 이는 보안 코드 관련한 지식의 부족일 수도 있고 다른 직면한 과제로 크게 신경쓰기 힘들 수도 있다.
 
### 목차
 
- Preventing memory-corruption vulnerabilities
 
- Preventing injection attacks
 
- Secure firmware updates
 
- Securing sensitive information
 
- Hardening embedded frameworks
 
- Securing third-party code and components
 
### 1. Preventing memory-corruption vulnerabilities
- C언어에서는 bound 문제로 인해 많은 메모리 corruption 의 문제가 발생할 가능성이 높다.
 
- 이러한 메모리 corruption 에 취약한 API 함수로는 문자열 함수의 strcat, strcpy, sprintf, scanf, gets 함수들이 있다. 
![img](https://upload.wikimedia.org/wikipedia/commons/4/4f/Stack_Overflow_2.png){:height="500px" width="350px"}
![img2](https://upload.wikimedia.org/wikipedia/commons/c/c3/Stack_Overflow_4.png){:height="500px" width="400px"}
example code
```c
int main(void){
    char buff[15];
    int pass = 0;

    printf("\n Enter the password : \n");
    gets(buff);

    if(strcmp(buff, "heeseung"))
        printf("\n Wrong Password \n");
    else{
        printf("\n Correct Password \n");
        pass = 1;
    }

    if(pass)
        printf ("\n Root privileges given to the user \n");
}
```

```sh
Enter the password : 

heeseung

Correct Password

Root privileges given to the user 
```

```sh
Enter the password : 

hhhhhhhhhhhhhhhhhhhh

Wrong Password

Root privileges given to the user 
```
=> overflow 된 "hhhhhhhh" 가 pass 변수의 값에 영향을 주어 원하지 않는 결과가 도출할 수 있다.

#### 1-1. How to do it??

리눅스에서는 C/C++ 코드파일의 이러한 오류를 찾아주는 유용한 기능을 가지고 있다.
 
- !!!!! 제목 확인 !!!!!
    - 메모리 Corruption 이 자주 발생하는 함수가 쓰이는 지 확인하는 방법이다. 이 명령어를 활용하서 어디서 memory corruption 이 취약한 함수가 쓰였는지 확인하여 주의하면 좋다.
```sh
grep -E '(strcpy|strcat|sprintf|strlen|memcpy|fopen|gets)' code.c
```

- FlawFinder 활용
    - 플로파인더는 C/C++ 코드를 검토하고 위험 수준에 따라 분류한 잠재적인 취약점을 보고하는 오픈소스툴이다. 대표적인 공개 정적 분석도구로 버퍼 오버플로우 문제, 포맷 문자열 문제, 레이스 조건, 형편 없는 무작위 숫자 획득 등의 알려진 위험이 있는 언어 함수의 내장 데이터베이스를 이용한다.
```c
#include <stdio.h>
#include <unistd.h>

#define BUFSIZER1   512
#define BUFSIZER2   ((BUFSIZER1/2) - 8)

int main(int argc, char **argv) {  
    char *buf1R1;
    char *buf2R1;
    char *buf2R2;
    char *buf3R2;

    buf1R1 = (char *) malloc(BUFSIZER1);
    buf2R1 = (char *) malloc(BUFSIZER1);

    free(buf2R1);

    buf2R2 = (char *) malloc(BUFSIZER2);
    buf3R2 = (char *) malloc(BUFSIZER2);

    strncpy(buf2R1, argv[1], BUFSIZER1-1);
    free(buf1R1);
    free(buf2R2);
    free(buf3R2);
}
```

```sh
Flawfinder version 1.31, (C) 2001-2014 David A. Wheeler. Number of rules (primarily dangerous function names) in C/C++ ruleset: 169 Examining a.c
 
FINAL RESULTS:
 
uaf.c:21: [1] (buffer) strncpy: Easily used incorrectly; doesn’t always \0-terminate or check for invalid pointers (CWE-120).
 
ANALYSIS SUMMARY:
 
Hits = 1 Lines analyzed = 27 in approximately 0.01 seconds (2470 lines/second) Physical Source Lines of Code (SLOC) = 19 Hits@level = [0] 0 [1] 1 [2] 0 [3] 0 [4] 0 [5] 0 Hits@level+ = [0+] 1 [1+] 1 [2+] 0 [3+] 0 [4+] 0 [5+] 0 Hits/KSLOC@level+ = [0+] 52.6316 [1+] 52.6316 [2+] 0 [3+] 0 [4+] 0 [5+] 0 Minimum risk level = 1 Not every hit is necessarily a security vulnerability. There may be other security vulnerabilities; review your code! See ‘Secure Programming for Linux and Unix HOWTO’ (http://www.dwheeler.com/secure-programs) for more information.
```
진행 과정 중 취약 부분에 대해서 Output 으로 알려준다.
```c
if(sizeof(buf2R1)>sizeof(argv[1]))
    strncpy(buf2R1, argv[1], BUFSIZER2);
```
```sh
(buffer) strncpy: Easily used incorrectly; doesn’t always 
```
하지만 위의 코드를 수정하여 직관적으로 봐도 충분히 buffer overflow 가 발생하지 않을 환경임에도 strncpy 라는 버퍼가 발생할 확률이 높은 함수가 코드내에만 있는지 확인하는 용도 정도로 사용하면 좋다.

- 사용하고자 하는 code에 취약한 점이 발견되면 바로 대체 함수나 다른 알고리즘으로 바꿔야 한다.
예를 들어 getS() 함수와 같이 버퍼 길이를 검사하지 않는 io함수를 buffer overflow 를 예방할 수 있는 fgets() 함수로 대체하여 사용할 수 있다.
```c
void input(){
    char buff[10];
    gets(buff); // 만약 입력문자열이 15bytes라면 오버플로우 발생!!
}
```
 
```c
void input(){
    char buff[10];
    fgets(buff, 10, stdin); // 만약 입력 문자열이 15bytes라면 10bytes만 buff에 입력되고 남은 5bytes는 다음 입력함수로 넘겨버린다. 따라서 오버플로우를 예방할 수 있다.
}
```

마찬가지로 snprintf(), strlcpy(), strlcat() 을 사용하여 버퍼 오버플로우의 위험성을 방지할 수 있다.(몇몇의 OS에 따라 함수사용의 제약이 있을 수 있음.)
 
```c
int snprintf(char *buffer, size_t n, const char *format-string, argument-list);
```
```c
size_t strlcpy(char * dest, const char * src, size_t size);
```
```c
size_t strlcat(char *dest, const char *src, size_t size);
```
 
### 2. Preventing injection attacks
- injection attacks 웹 애플리케이션과 특히 IoT에 가장 취약한 부분 중 하나이다. injectiong attacks 에는 다음 4가지가 있다.
    1. OS System command injection 
    2. cross-site scripting (JavaScript injection)
    3. SQL injection 
    4. log injection
- IoT와 임베디드 시스템에서 대부분 OS Command injecetion 이다.
 
#### 2-1. How to do it
- Firmware 는 exec() system() 와 같거나 유사한 명령어로 OS call 를 실행하여 버퍼오버플로우를 유발할 수 있다. 따라서 이러한 공격을 예방해야 하며 이러한 참고는 다음과 같다.

```c
#include <stdlib.h>
 
void func(void) {
  system("rm ~/.cfg");
}
```
- 취약한 코드를 방식하기 위해 system() 함수 대신에 unlink() 함수를 사용한다.
  
```c
#include <pwd.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
void func(void) {
const char *file_format = "%s/.cfg";
size_t len;
char *pathname;
struct passwd *pwd;
 
pwd = getpwuid(getuid());
if (pwd == NULL) {
   /* Handle error */
}
 
len = strlen(pwd->pw_dir) + strlen(file_format) + 1;
pathname = (char *)malloc(len);
if (NULL == pathname) {
   /* Handle error */
}
int r = snprintf(pathname, len, file_format, pwd->pw_dir);
if (r < 0 || r >= len) {
   /* Handle error */
}
if (unlink(pathname) != 0) {
   /* Handle error */
}
 
free(pathname);
}
```
unlink 는 연결 계수를 1 감소하는 시스템 호출로 만약 unlink 호출되었을 대 연결 계수(symlink)를 확인하고 연결계수가 0이면 이 대 파일을 삭제한다. 이 때 unlink() 는 symlink 만 조절하므로 파일이나 디렉토리의 영향을 주지 않는다.
 
- 이외에도 injection attacks 를 예방하기 위해 참고사항 들이다. 
    + 가능하면 OS 호출을 직접 호출하지 않도록 합니다. 필요한 경우 허용한 명령을 화이트리스트에 추가하고 실행 전에 입력 값을 확인합니다.
    + 운영 체제에 전달할 수 있는 사용자 기반 문자열(예: {1:ping -c 5})에 대해 숫자 대 명령 문자열의 조회 맵을 사용하십시오.
    + code base에 대한 정적 코드 분석을 수행하고 os.system()과 같은 언어가 OS 명령어일 때 알림을 보냅니다.
    + 모든 사용자 입력을 신뢰할 수 없는 것으로 간주하고, 사용자에게 다시 렌더링된 데이터의 문자를 출력에서 인코딩합니다(예: Convert &amp; to &amp;, Convert <를 &lt로 변환, >를 &gt로 변환 등).
    + XSS의 경우 X-XSS-Protection 및 Content- Security-Policy와 같은 HTTP 응답 헤더를 적절한 지시어로 사용합니다.
    + 프로덕션 펌웨어 빌드(예: http://example.com/command.php))에서 명령 실행을 사용하는 디버그 인터페이스가 비활성화되었는지 확인합니다.
 
## Reference
- <https://www.embedded.com/firmware-security-preventing-memory-corruption-and-injection-attacks/>