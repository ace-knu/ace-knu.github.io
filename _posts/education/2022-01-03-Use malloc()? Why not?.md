---
layout: single
title: "Use malloc()? Why not?"
categories:
    - education
tags: 
    - C/C++
    - Linux
    - Embedded
---

- 메모리 동적할당(memory allocation) 이란?  
    컴퓨터 프로그래밍에서 실행시간동안 사용할 메모리 공간을 할당하는 것. 사용이 끝나면 운영체제가 쓸수 있도록 반납하고 다움에 다시 메모리공간 할당 요청이 오면 재할당 받을 수 있다  
    일반적으로 스택영역에 사용되는 정적메모리와 달리 힙영역을 사용한다.

![img1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbSd1lI%2FbtqH2dJqR3X%2FizWltqNIz6dRDMfIwDtRK1%2Fimg.png)


![img2](https://blog.kakaocdn.net/dn/bvm3fx/btqHYIQTKk6/OviCnDz2azXaJdUCdNGO6K/img.png)

## 임베디드에서 동적메모리 할당을 가급적 자제해야 하는 이유
1) malloc() 은 reentrant function 이 아니기 때문에 멀티 thread 환경에서 사용하지 쉽지 않다. malloc은 global heap 에서 호출되기 때문이다. 
  - __Thread-safety__  
 여러 Thread에서 동시에 실행해도 문제 없는 함수, 다시 말해 여러 Thread가 같은 함수를 동시에 실행할 경우 Thread 내에 함수가 이용하는 Thread 간 공유자원문제이다. 공유자원을 Lock같은 동기화 기법으로 보호하여 공유 자원의 무결성을 보장하는 함수를 Thread-safe 함수라고 한다.

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int global_x = 0;

int thread_safe_function() {
    pthread_mutex_lock(&mutex);
    ++global_x;
    pthread_mutex_unlock(&mutex);
    return global_x;
}
```

  - __reentrant__  
  Thread-safe 함수와 마찬가지로 여러 Thread에서 동시에 실행이 가능하지만 Thread 간 공유 자원를 이용하지 않는 함수. 공유 변수를 이용하지 않기 때문에 각 Thread는 언제나 같은 호출 결과를 얻을 수 있다. 이러한 성질을 Reentrancy(재진입 가능한) 하다라고 표현하기 때문에 Reentrant 함수라고 한다.

```c
int reentrant_function(){
    int local_x = 0;
    ++local_x;
    return global_x;
}
```

__=> malloc 함수 내의 global lock 기능이 있음. 따라서 thread-safety 을 보장한다.__
<br/>
   
 ```c
malloc();           //initial call
lock(memory_lock);  //acquire lock inside malloc implementation
signal_handler();   //interrupt and process signal
malloc();           //call malloc() inside signal handler
lock(memory_lock);  //try to acquire lock in malloc implementation

// DEADLOCK!  We wait for release of memory_lock, but 
// it won't be released because the original malloc call is interrupted
```  

하지만 만약 malloc()을 사용하다가 signal_handler() 에서 interrupt 가 발생하여 실행이 멈출 때 교착상태에 빠질 수 있다. 따라서 malloc()은 reentrant 함수가 아니다.
<br/>

2) **메모리 블록을 할당하는데 걸리는 시간이 일정하지 않아서 성능을 예측하기 쉽지 않고 많은 시간이 소요된다.**
- 동적할당

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

void main(){
	clock_t t1, t2;
	
	t1 = (double)clock();
	int *p = (int*)malloc(100000);
	t2 = (double)clock();
	printf("%f\n", (float)(t2-t1));
	free(p);

	printf("f\n", (float)(t4-t3));
}
```

```sh
time : 21
time : 19
time : 23
```

- 정적할당

```c
clock_t t1, t2;
t1 = (double)clock();
int p[100000];
t2 = (double)clock();
printf("time : %f\n", (float)(t2-t1));
```

```sh
time : 3.0
time : 4.0
time : 2.0
```

동적할당이 느린 이유는 메모리 재배치 때문인데, 가령 메모리가 4bytes의 여유분이 남았는데 int형 변수를 동적으로 할당했을 때 이전 배치된 메모리들의 위치 때문에 fragmentaion 이 발생하는 데 이 때 운영체제는 4bytes를 만들기 위해 메모리의 요소를 자동으로 재배치 한다.
<br/>
![이미지](https://www.gatevidyalay.com/wp-content/uploads/2018/11/Contiguous-Memory-Allocation-Translating-Logical-Address-into-Physical-Address-Diagram.png)

<hr/>

3) __Heap 메모리 부족, heap fragmentation에 의해 메모리 할당에 실패할 가능성이 있음__  

- __Heap fragmentation__ : RAM에서 메모리의 공간이 작은 조각으로 나뉘어져 사용가능한 메모리가 충분히 존재하지만 할당(사용)이 불가능한 상태

- __Internal fragmentation__ : 메모리를 할당할 때 프로세스가 필요한 양보다 더 큰 메모리가 할당되어서 프로세스에서 사용하는 메모리 공간이 낭비 되는 상황

![이미지](https://d3e8mc9t3dqxs7.cloudfront.net/wp-content/uploads/sites/11/2020/05/Fragmentation3.png)  

- __External fragmentation__ : 메모리가 할당되고 해제되는 작업이 반복될 때 작은 메모리가 중간중간 존재하게 된다. 이 때 중간중간에 생긴 사용하지 않는 메모리가 많이 존재해서 총 메모리 공간은 충분하지만 실제로 할당할 수 없는 상황

![이미지](https://d3e8mc9t3dqxs7.cloudfront.net/wp-content/uploads/sites/11/2020/05/Fragmentation4.png)






----
## Solution??
1. re-entrant 문제의 경우 여러 Thread 를 호출하는 경우에만 생기는 문제이다. 이 때 메모리 할당과 해제만 수행하는 단일작업을 만들어서 해결할 수 있다. 따라서 다른 쓰레드 작업 수행 중 메모리가 필요하다면 할당/해제를 수행하는 작업에 요청하여 해결할 수 있다. 
2. 많은 Application 들은 메모리 성능과 실행시간의 정확한 예측은 필요 없을 수도 있다.
3. 할당이 실패한 경우 문제가 될 수 있지만 exception 처리를 통해 관리할 수 있다. heap 영역의 메모리가 부족하거나 heap fragmentation에 의해 메모리가 할당되지 않을 때 NULL 값이 반환되는데 이 반환값을 틈틈히 모니터링 하여 관리할 수 있다.
```c
a = malloc(200);
if(a==NULL)
    exception(); ...
```

-  메모리 할당과 해제를 순차적으로 이루어지도록 하는 방법으로 heap fragmentation 의 문제를 해결 할 수 있다. 하지만 이건 확실한 해결책이 될 수 없다.
```c
a = malloc(1000);
b = malloc(100);
c = malloc(5000);
...
free(c);
free(b);
free(a);
```

많은 Application 은 malloc의 최대 장점인 언제든 메모리를 불러오고 해제할 수 있는 그런 유연성을 꼭 필요로 하지 않는다. 따라서 필요한 고정된 메모리를 할당하여 작성하는 것이 가장 간단하면서 좋은 해결책이다.  

하지만 고정된 메모리 할당 역시 문제점이 있다. 분명 사용하지 않는 메모리가 발생하고 이는 메모리 관리 측면에서 효율성을 떨어 뜨릴 수 있다.

__Memory pool__  
마지막으로 위의 두가지 문제점을 최소화 하면서 유연성을 얻는 방법은 할당하고자 하는 메모리 크기에 따라 pool을 만들어 필요로 하는 메모리 크기를 고려하여 블록을 선택하는 방법이다.   
이 때 Memory pool 의 크기는 16bytes, 32bytes 64byes, 128bytes... 등을 사용하는게 유용하다. 

![이미지](https://www.embedded.com/wp-content/uploads/2021/02/02cw_f2.png)

물론 Memory pool 도 internal fregmentation 의 문제를 해결할 수는 없다. 가령 65bytes 을 할당받고자 할 때 128bytes pool을 할당해야 하고 이 때 63bytes의 쓸데 없이 메모리가 발생한다. 하지만 이 정도는 유연성을 얻기 위한 최소한의 비용이라고 생각하면 좋을 거 같다.
```c
typedef struct poolFreed{
	struct poolFreed *nextFree;
} poolFreed;

typedef struct {
	uint32_t elementSize;
	uint32_t blockSize;
	uint32_t used;
	int32_t block;
	poolFreed *freed;
	uint32_t blocksUsed;
	uint8_t **blocks;
} pool;
  
void poolInitialize(pool *p, const uint32_t elementSize, const uint32_t blockSize){
	uint32_t i;

	p->elementSize = max(elementSize, sizeof(poolFreed));
	p->blockSize = blockSize;
	
	poolFreeAll(p);

	p->blocksUsed = POOL_BLOCKS_INITIAL;
	p->blocks = malloc(sizeof(uint8_t*)* p->blocksUsed);

	for(i = 0; i < p->blocksUsed; ++i)
		p->blocks[i] = NULL;
}

void poolFreePool(pool *p){
	uint32_t i;
	for(i = 0; i < p->blocksUsed; ++i) {
		if(p->blocks[i] == NULL)
			break;
		else
			free(p->blocks[i]);
	}

	free(p->blocks);
}

#ifndef DISABLE_MEMORY_POOLING
void *poolMalloc(pool *p){
	if(p->freed != NULL) {
		void *recycle = p->freed;
		p->freed = p->freed->nextFree;
		return recycle;
	}

	if(++p->used == p->blockSize) {
		p->used = 0;
		if(++p->block == (int32_t)p->blocksUsed) {
			uint32_t i;

			p->blocksUsed <<= 1;
			p->blocks = realloc(p->blocks, sizeof(uint8_t*)* p->blocksUsed);

			for(i = p->blocksUsed >> 1; i < p->blocksUsed; ++i)
				p->blocks[i] = NULL;
		}

		if(p->blocks[p->block] == NULL)
			p->blocks[p->block] = malloc(p->elementSize * p->blockSize);
	}
	
	return p->blocks[p->block] + p->used * p->elementSize;
}

void poolFree(pool *p, void *ptr){
	poolFreed *pFreed = p->freed;

	p->freed = ptr;
	p->freed->nextFree = pFreed;
}
#endif

void poolFreeAll(pool *p){
	p->used = p->blockSize - 1;
	p->block = -1;
	p->freed = NULL;
}
```

## Reference
- <https://www.embedded.com/use-malloc-why-not/?oly_enc_id=1916E7570689H8V>
- <https://github.com/jobtalle/pool>

