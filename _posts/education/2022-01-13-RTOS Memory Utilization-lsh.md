---
layout: single
title: "RTOS Memory Utilization"
categories:
    - education
tags: 
    - C/C++
    - Linux
    - RTOS
---

## RTOS란?
- Computing system whose specification includes both logical and temporal correctness
    - Logical correctness: produces correct outputs
    - Temporal correctness: produces outputs at the right time
- 주로 임베디드 시스템에서 사용
    - 특정 작업만 하도록 살계된 경우가 많고 시간안에 처리하는 것이 중요한 일에 많이 사용되기 때문
- application code에 사용될 수 있었던 시간이나 메모리를 사용하므로 OS는 궁극적으로 오버헤드임
- 리소스가 제한되어 있으므로 RTOS가 메모리를 사용하는 방법과 메모리 사용량을 정확하게 파악하기 힘듦



## RTOS의 메모리 사용 방법
### 1. RAM, ROM만 존재
- 성능 향상을 위해 부팅 시 ROM에서 RAM으로 코드와 데이터를 복사한 다음 RAM을 활용함
    - 일반적으로 RAM이 ROM보다 액세스 속도가 더 빠르므로 효과적
    - RTOS 메모리 사용량 고려 시 ROM, RAM 크기 뿐만 아니라 RAM 복사 가능성 또한 고려해야함

### 2. on-chip RAM과 사용가능한 외부 메모리가 있는 경우
- on-chip storage의 접근 속도가 더 빠르므로 RTOS code나 data가 해당 storage에 저장되어있는 것이 더욱 유리
    - 결국 이 성능이 전제 애플리케이션의 영향을 미침
- code/data는 캐시 메모리에 저장되어 더 높은 성능을 보일 수도 있음



## RTOS의 메모리 사용량 파악이 힘든 이유
### 1. 코드 크기 문제

다양한 요인이 코드 크기에 영향을 줄 수 있음


#### 1.1 CPU Architecture
- RTOS 메모리 공간에 큰 영향을 미침
- 사용하려는 CPU 용도로 구현된 코드들의 수치 정보만을 가지고 코드의 크기를 비교할 수 있음
    - 동일한 코드여도 PowerPC의 코드 크기와 ARM의 코드 크기가 매우 다를 수 있음


#### 1.2 컴파일러 최적화
- 컴파일러에 적용되는 최적화 옵션이 코드의 크기와 실행 속도 모두에 영향을 미칠 수 있음
    - 최고 성능을 위해 빌드된 코드는 크기가 커질 것이며 더 작게 최적화된 코드는 더 느릴 것
- 보통 RTOS는 크기보다는 성능을 위해 구축될 가능성이 높으나 제품의 사이즈를 강조할 경우에는 다른 선택을 할 수도 있음


#### 1.3 RTOS의 구성

![f1](https://www.embedded.com/wp-content/uploads/contenteetimes-images-design-embedded-2015-09-walls-rtosscalability.jpg)

- RTOS는 대부분 구성이 간단하며 이는 RTOS 크기를 크게 변화시킬 수 있음
- 대부분의 RTOS 제품은 확장 가능하므로 메모리 살치 공간은 응용 프로그램에서 사용하는 실제 서비스에 따라 결정됨
- 필요한 서비스에 따라 확장성의 세분화는 제품마다 다름
    - RTOS의 object에 대한 지원이 필요한 경우 관련된 모든 서비스가 포함
    - 그래픽, 네트워킹 등 다른 옵션 또한 마찬가지임
    - 따라서 이러한 옵션의 필요성에 따라 코드의 크기에 영향을 미침


#### 1.4 Runtime Library
- 일반적으로 런타임 라이브러리는 RTOS와 함께 사용됨
- `1.3`의 경우와 동일하게, 특정 응용 프로그램의 필요에 따라 확장될 수 있음



### 2. 데이터 크기 문제
변수에 대한 기본 스토리지 용량과는 별개로, RTOS의 RAM 요구사항은 다음과 같은 요인으로 인해 비슷하게 영향을 받을 수 있음


#### 2.1 컴파일러 최적화
- `1.2`와 마찬가지로 컴파일러 최적화는 데이터 크기에도 영향을 줌
- 더 많이 압축될수록 데이터 크기는 더 작지만 액세스하는 데 더 많은 시간과 instructions가 필요

#### 2.2 RTOS 개체
- 애플리케이션에서 사용하는 RTOS object(task, semaphores, mailboxes etc.)는 각 개체마다 약간의 RAM공간이 필요하므로 RTOSRAM사용에 영향을 미침


#### 2.3 스택
- 일반적으로 OS는 스택을 가지며, 모든 작업에는 자체 스택이 있음
    - 이들은 모두 RAM에 저장됨
- 이 공간의 할당은 RTOS마다 다르게 수행될 수는 있으나 무시될 수는 없음


#### 2.4 동적 메모리
- OS가 프로그램에게 메모리를 실시간으로 할당할 때는 동적 할당을 사용함
- 메모리 동적 할당이 가능하고, 응용 프로그램에서 해당 기능을 사용해야한다면 이를 위한 충분한 크기의 메모리 풀이 할당되어야 함

## 정적 및 동적 RTOS 구성
- 초기 RTOS products는 build 시, 정적으로 configuration을 수행해야 했으나  현재는 동적으로 생성(및 파괴)하는 것이 일반적
    - 정적 구성을 허용하는 RTOS를 찾는 것이 더 드물어짐

- 정적으로 구성된 RTOS는 ROM의 RTOS 개체에 대한 대부분의 데이터를 보관
    - 일부 정보는 실행 중에 변경되어 RAM에 복사해야하나, 초기화가 필요
    - 다른 개체는 런타임에 추가 RAM 공간이 필요
    
- 동적으로 구성된 RTOS는 모든 개체 데이터를 RAM에 보관하며 ROM에는 전혀 보관하지 않음
    - 개체 생성 및 삭제를 수행하기 위한 추가 서비스 호출이 있기 때문에 ROM 공간에 상당한 히트가 있음

- 따라서 어떤 RTOS의 메모리 사용과 사용량을 파악하는 데에는 복합적인 요소룰 고려하는 것이 필요



## 부록

### 1. GPOS와 RTOS 차이
![f2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbVpCpz%2Fbtq8dtAsXac%2FhpUEiKwgqDMewRXEaadb2k%2Fimg.png)

### 2. CISC vs RISC
#### CISC
![f3](https://t1.daumcdn.net/cfile/tistory/182B1F4A514EFB8C0D)
- 연산에 처리되는 복잡한 명령어들을 수백 개 이상 탑재하고 있는 프로세서
- 복잡하므로 해석하는데 오랜 시간이 걸리며, 명령어 해석에 필요한 회로도도 복잡함
    - 병렬처리 불가능
- 하지만 프로그래밍 작업이 간단하며 다양한 주소 지정 모드를 사용함
- 작은 코드 크기, 단위 시간 동안 높은 사이클을 가짐
- ex) Intel X86, AMD64  etc.

#### RISC
![f4](https://t1.daumcdn.net/cfile/tistory/01272A4B514EFB6302)
- Reduced Instruction Set Computer
- 복잡한 80% 가량의 명령어를 제거하여 사용 빈도가 높은 명령어 위주로 20%의 명령어를 HW화하여 처리 속도를 향상시킨 프로세서
- 컴퓨터 실행 속도를 높이기 위해 복잡한 처리는 소프트웨어에게 맡기는 방법을 채택
- CPU에서 수행하는 모든 동작의 대부분이 몇 개의 명령어만으로도 가능하다는 사실을 전제로 설계됨
- 모든 명령어의 길이가 같아 병렬처리 가능
- CISC에 비해 상대적으로 많은 범용 레지스터를 사용함
- 단위 시간 동아 낮은 사이클 수, 큰 코드 크기를 가짐
- ex) ARM, MIPS, POWERPC etc.

### [3. Memory pool](https://www.ibm.com/docs/en/i/7.1?topic=concepts-memory-pools)


### 4. Reference
- <https://www.embedded.com/rtos-memory-utilization/>
