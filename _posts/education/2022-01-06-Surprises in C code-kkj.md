---
layout: single
title: "Surprises in C code"
categories:
    - education
tags: 
    - C/C++
    - Linux
---

## 1. For Loop 와 While Loop
#### For Loop
```c
for(int i=0;i<10;++i){
    
}
```

#### While Loop
```c
int i=0;
while(i<10){
    ++i;
}
```
- For 문과 While 문은 동일한 표현이 가능함
- 다음과 같이 For Loop에 다중 Statement도 표기 가능함
```c
for(int i=0,j=10 ;i<j; i++, j--){
    ...
}
```

## 2. i++ and ++i
- i++ : 변수에 i 값 저장 후 증가 
- ++i : 증가된 값이 저장됨

#### 성능 차이??
- C언어가 만들어진 초창기 시절에는 컴파일러가 좋지 않아서 Prefix-Expression 선호했었음

```c
// Prefix Expression
for(int i=0;i<10;++i){
    ...
}
``` 

#### prefix 테스트 코드

```c
#define MAX 1000000000
int arr[MAX];
int main(){
    for(int i=0;i<MAX;++i){
        arr[i] = i;
    }  
}
```

#### 어셈블리 코드 (gcc -O0 -S loop1.c)

- Prefix / Postfix 동일한 어셈블리 결과물을 출력함
```
main:
.LFB0:
    .cfi_startproc
    endbr64
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    movl    $0, -4(%rbp)
    jmp .L2
.L3:
    movl    -4(%rbp), %eax
    cltq
    leaq    0(,%rax,4), %rcx
    leaq    arr(%rip), %rdx
    movl    -4(%rbp), %eax
    movl    %eax, (%rcx,%rdx)
    addl    $1, -4(%rbp)
.L2:
    cmpl    $999999999, -4(%rbp)
    jle .L3
    movl    $0, %eax
    popq    %rbp
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc
```

#### For Loop 부분

```
movl    $0, -4(%rbp)
    jmp .L2
.L3:
    ...
    ...
    addl    $1, -4(%rbp)
.L2:
    cmpl    $999999999, -4(%rbp)
    jle .L3
```

#### 포인터에서의 Prefix / Postfix

```c
int arr[5] = {0,0,0,0,0}, arr2[5] = {0,0,0,0,0};
int *ptr = arr, *ptr2 = arr2;

for(int i=0;i<5;++i){
    *(ptr++) = i;
    *(++ptr2) = i
}
```
![image](/assets/img/education/RST.png)

## 3. Multiple assignment
```c
a = b = c = 7; 
d = 7; e = 7; f = 7;
```
```c
a = b = c = 7; 
// a = (b = (c = 7))
```
- assignment는 right to left로 판별
![image](/assets/img/education/OP.png)

- 변수가 아닌 레지스터를 설정하는 경우 Multiple Assignment가 적합하지 않을 수 있음

- 변수의 데이터 타입이 다를 경우 문제 발생

```c
    int b;
    double a, c;
    a = b = c = 5.2; // 5 5 5.2
```
![DEBUG](/assets/img/education/DEBUG.png)

```c
    int16_t a, c;
    int8_t b;
    a = b = c = 129; // -127 -127 129
```

### Multiple Assignment 테스트
#### 테스트 코드
```c
    int a, b, c,
        d, e, f;
    a = b = c = 7;
    printf("%d", c);
    d = 8, e = 8, f = 8;
    printf("%d", f);
```

#### 어셈블리 파일 (gcc -S ...)
```
main:
.LFB0:
    .cfi_startproc
    endbr64
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register 6
    subq    $32, %rsp
    movl    $7, -24(%rbp)
    movl    -24(%rbp), %eax
    movl    %eax, -20(%rbp)
    movl    -20(%rbp), %eax
    movl    %eax, -16(%rbp)
    movl    -24(%rbp), %eax
    movl    %eax, %esi
    leaq    .LC0(%rip), %rdi
    movl    $0, %eax
    call    printf@PLT
    movl    $8, -12(%rbp)
    movl    $8, -8(%rbp)
    movl    $8, -4(%rbp)
    movl    -4(%rbp), %eax
    movl    %eax, %esi
    leaq    .LC0(%rip), %rdi
    movl    $0, %eax
    call    printf@PLT
    movl    $0, %eax
    leave
    .cfi_def_cfa 7, 8
    ret
    .cfi_endproc

    ...
    ...
```

#### Multiple Assignment
```c
    a = b = c = 7;
    printf("%d", c);
```

```asm 
    movl    $7, -24(%rbp)   // c = 7
    movl    -24(%rbp), %eax
    movl    %eax, -20(%rbp) // b = c
    movl    -20(%rbp), %eax
    movl    %eax, -16(%rbp) // a = b
    movl    -24(%rbp), %eax 
    movl    %eax, %esi
    leaq    .LC0(%rip), %rdi
    movl    $0, %eax
    call    printf@PLT      // print c
```

#### Simple Assignment
```c
    d = 8, e = 8, f = 8;
    printf("%d", f);
```

```asm
    movl    $8, -12(%rbp) // d = 8
    movl    $8, -8(%rbp)  // e = 8
    movl    $8, -4(%rbp)  // f = 8
    movl    -4(%rbp), %eax
    movl    %eax, %esi
    leaq    .LC0(%rip), %rdi
    movl    $0, %eax
    call    printf@PLT // print f
```

## 4. Reference
[Surprises in C code](https://www.embedded.com/surprises-in-c-code/?oly_enc_id=1916E7570689H8V)