---
layout: single
title: "Google Test"
categories:
    - education
tags: 
    - C/C++
    - Automotive SW
---

## 들어가기 앞서...

- [GoogleTest]([https://github.com/google/googletest) ⇒ Google의 C++ 유닛 테스트 프레임워크

## 1. GoogleTest가 쓰이는 오픈소스 프로젝트

- Chromium project (크롬 브라우저와 크롬 OS)
- OpenCV
- Android Open Source project
- etc..

## 2. Google Test를 사용하는 이유

EX) 여러가지 자료구조를 설계해야 하는 상황

```c
//Linked List
struct node{
	int num;
	struct node *next;
};
typedef struct node NODE;
typedef NODE* LINK; 

// InsertNode : front - end
void InsertNode(LINK front, LINK end){
	front->next = end;
}

// DeleteNode : Delete All LINK in target Node
void Deletenode(LINK node){
	free(node->next);
}

// PrintNode : print All element in each node
void PrintNode(LINK node){
	while(!node->next)
		printf("%d ", node->num);
	puts("");
}
```

Single Linked List 미완성 코드에서
- InsertNode 함수
    - front, end == NULL인 상황 고려 X
- DeleteNode 함수
    - 타겟 노드가 3개 이상 NODE를 가진 상황에서  free 했을 때 메모리 누수 발생 가능성 O

등 고려하지 않은 상황이 많으며 아직 해당 코드가 올바르게 동작하는 지 테스트를 해볼 필요가 있다.

#### Main 문에서 테스트

```c
void main(){
    LINK test;
    test->num = 1;
    test->LINK = NULL;

    PrintNode(test);
}
```

- 프로그램이 커질수록 테스팅 비용이 증가한다
- CreateNode() 새로운 함수 추가 :  PrintNode() → InsertNode() → DeleteNode() 정상 동작 확인

#### Unit Test의 도입

- 문제점 발견이 용이
	- 프로그램을 나눠서 테스트 
	ex) Linked List 관련 코드 + Queue 관련 코드
	- 상황을 나눠서 테스트 
	ex) 노드가 1개일 때 상황 / 2개일 때 / 3개일 때 ...
- 변경이 쉬움 
	- 코드 리팩토링 용이
	⇒ 검증하고자 하는 부분(메소드 등등)만 빼내서 테스트 가능
	- Regression Test
	⇒ CreateNode 함수 변경시 InsertNode 함수에 영향 가는지 테스트
- 통합이 간단 
	- 유닛 테스트 → 통합 테스트
	ex) 위 예시에서 DeleteNode 함수 메모리 해제 잘 했는 지 테스트
	- 코드 재사용시 용이
	ex) Linked List를 통해 Tree 구현

## 3. Google Test의 특징
- 테스트와 객체 분리 : 한 테스트로 여러 객체 테스트 가능 
- 다양한 OS 및 컴파일러 지원 : Linux /  Windows / Mac
- pthread 라이브러리를 이용하기 때문에 Thread-Safe 보장 받음

## 4. Terminology
1. Assertion : 조건이 참인지 체크하는 문장
Return 값 → success / nonfatal failure / fatal failure
fatal failure 일때 abort
```c
EXPECT_EQ(fac(0), 0);
```
2. Test : Assertion을 정의하기 위한 Case
```c
TEST(factorial_test, ZeroInput){
	EXPECT_EQ(fac(0), 0);
}
```
3. Test suite : 1개 이상의 Test 그룹
그룹 내의 Test들이 공유 자원을 사용할 경우 test fixture class 이용
```c
TEST(factorial_test, ZeroInput){
	EXPECT_EQ(fac(0), 0);
}
TEST(factorial_test, PosInput){
	EXPECT_EQ(fac(1), 1);
	EXPECT_EQ(fac(2), 2);
	EXPECT_EQ(fac(3), 6);
	EXPECT_EQ(fac(4), 24);
	EXPECT_EQ(fac(5), 120);
}
```

4. Test program : 1개 이상의 Test Suite  그룹 ⇒ ```test.cc```
#### Test P → Test suite → Test → Assertion
![TEST process](/assets/img/Automotive/TEST_p.jpg)

## 5. Google Test 예시

#### 구성

- fac.h
- fac.cc
- test.cc
- Makefile

| Test P  | Test Suite     | Test      | Assertion |
|---------|----------------|-----------|-----------|
| test.cc | factorial_test | ZeroInput | EXPECT_EQ |
|         |                | PosInput  | EXPECT_EQ |
|         | fibonacci_test | ZeroInput | EXPECT_EQ |
|         |                | PosInput  | EXPECT_EQ |

#### 코드

```c
// fac.h
int fac(int n);
```

```c
// fac.cc
#include "fac.h"

int fac(int n){
	if(n<1) return 1;
	else return(n * fac(n-1));
}
```

```c
// test.cc
#include <stdio.h>
#include <gtest/gtest.h>
#include "fac.h"

TEST(factorial_test, ZeroInput){
	EXPECT_EQ(fac(0), 0);
}

TEST(factorial_test, PosInput){
	EXPECT_EQ(fac(1), 1);
	EXPECT_EQ(fac(2), 2);
	EXPECT_EQ(fac(3), 6);
	EXPECT_EQ(fac(4), 24);
	EXPECT_EQ(fac(5), 120);
}

int main(int argc, char **argv){
	::testing::InitGoogleTest(&argc, argv);
	return RUN_ALL_TESTS();
}
```

```bash
// Makefile
.PHONY: test
GTEST_DIR=googletest/googletest

test:
	g++ -o test fac.cc test.cc -isystem ${GTEST_DIR}/include -L${GTEST_DIR}/build -pthread -lgtest
	./test
```

#### 결과
```
[==========] Running 2 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 2 tests from factorial_test
[ RUN      ] factorial_test.ZeroInput
[test.cc:8](http://test.cc:8/): Failure
Expected equality of these values:
fac(0)
Which is: 1
0
[  FAILED  ] factorial_test.ZeroInput (0 ms)
[ RUN      ] factorial_test.PosInput
[       OK ] factorial_test.PosInput (0 ms)
[----------] 2 tests from factorial_test (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] factorial_test.ZeroInput

1 FAILED TEST
make: *** [Makefile:6: test] Error 1
```
## 6. 테스트 프로그램 작성법

```c
// test.cc
#include <stdio.h>
#include <gtest/gtest.h> // -> gtest/gtest 참조
#include "fac.h"

//TEST(test_case_name, test_name)
TEST(factorial_test, ZeroInput){
	EXPECT_EQ(fac(0), 0);
}

TEST(factorial_test, PosInput){
	EXPECT_EQ(fac(1), 1);
	EXPECT_EQ(fac(2), 2);
	EXPECT_EQ(fac(3), 6);
	EXPECT_EQ(fac(4), 24);
	EXPECT_EQ(fac(5), 120);
}

int main(int argc, char **argv){
	::testing::InitGoogleTest(&argc, argv); // -> InitGoogleTest 메소드 필요
	return RUN_ALL_TESTS();
}
```

- ```::testing::InitGoogleTest(&argc, argv)``` : Command Line의 flag 파싱
- ```RUN_ALL_TESTS``` : gTest 결과 값 반환
	(1) gTest flag 저장
	(2) fixture 객체 생성
	(3) Setup 이후 테스트
	(4) fixture 객체 삭제
	(5) gTest flag 복원
	(6) 반복

## 7. Test Fixtures
공유 자원을 다양한 테스트에서 활용해야할 경우 사용
#### Fixture Form
```c++
TEST_F(TestFixtureName, TestName) {
  ... test body ...
}
```

#### Queue Type
```c++
template <typename E>  // E is the element type.
class Queue {
 public:
  Queue();
  void Enqueue(const E& element);
  E* Dequeue();  // Returns NULL if the queue is empty.
  size_t size() const;
  ...
};
```

#### test fixture
- ```::testing::Test``` 상속 받아서 사용
- Constructor 나 SetUp() 함수 필요
- Deconstructor TearDown() 통해 Release
```c++
class QueueTest : public ::testing::Test {
 protected:
  void SetUp() override {
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  // void TearDown() override {}

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};
```

#### Test Code
```c++
namespace{
	...
	TEST_F(QueueTest, IsEmptyInitially) {
	EXPECT_EQ(q0_.size(), 0);
	}

	TEST_F(QueueTest, DequeueWorks) {
	int* n = q0_.Dequeue();
	EXPECT_EQ(n, nullptr);

	n = q1_.Dequeue();
	ASSERT_NE(n, nullptr);
	EXPECT_EQ(*n, 1);
	EXPECT_EQ(q1_.size(), 0);
	delete n;

	n = q2_.Dequeue();
	ASSERT_NE(n, nullptr);
	EXPECT_EQ(*n, 2);
	EXPECT_EQ(q2_.size(), 1);
	delete n;
	}
	...
}

int main(int argc, char **argv){
	::testing::InitGoogleTest(&argc, argv);
	return RUN_ALL_TESTS();
}
```

#### 결과
```
[==========] Running 3 tests from 1 test suite.
[----------] Global test environment set-up.
[----------] 3 tests from QueueTestSmpl3
[ RUN      ] QueueTestSmpl3.DefaultConstructor
[       OK ] QueueTestSmpl3.DefaultConstructor (0 ms)
[ RUN      ] QueueTestSmpl3.Dequeue
[       OK ] QueueTestSmpl3.Dequeue (0 ms)
[ RUN      ] QueueTestSmpl3.Map
[       OK ] QueueTestSmpl3.Map (0 ms)
[----------] 3 tests from QueueTestSmpl3 (0 ms total)

[----------] Global test environment tear-down
[==========] 3 tests from 1 test suite ran. (0 ms total)
[  PASSED  ] 3 tests.

```

## 8. Assertion
#### - ```FAIL()``` / ```ADD_FAILURE()```
```c++
switch(expression) {
  case 1:
    ... some checks ...
  case 2:
    ... some other checks ...
  case 3:
	std::cout << ADD_FAILURE(); // fatal failure
  default:
    std::cout << FAIL(); // non-fatal failure
}
```

#### 8-1. Condition Test

|      Fatal assertion     |    Nonfatal assertion    |      Verifies      |
|:------------------------:|:------------------------:|:------------------:|
| ASSERT_TRUE(condition);  | EXPECT_TRUE(condition);  | condition is true  |
| ASSERT_FALSE(condition); | EXPECT_FALSE(condition); | condition is false |

#### 8-2. Binary Comparison
Compare Two Value

|     Fatal assertion    |   Nonfatal assertion   |   Verifies   |
|:----------------------:|:----------------------:|:------------:|
| ASSERT_EQ(val1, val2); | EXPECT_EQ(val1, val2); | val1 == val2 |
| ASSERT_NE(val1, val2); | EXPECT_NE(val1, val2); | val1 != val2 |
| ASSERT_LT(val1, val2); | EXPECT_LT(val1, val2); | val1 < val2  |
| ASSERT_LE(val1, val2); | EXPECT_LE(val1, val2); | val1 <= val2 |
| ASSERT_GT(val1, val2); | EXPECT_GT(val1, val2); | val1 > val2  |
| ASSERT_GE(val1, val2); | EXPECT_GE(val1, val2); | val1 >= val2 |

#### 8-3. String Comparison
Compare Two String

|        Fatal assertion       |      Nonfatal assertion      |                         Verifies                         |
|:----------------------------:|:----------------------------:|:--------------------------------------------------------:|
| ASSERT_STREQ(str1,str2);     | EXPECT_STREQ(str1,str2);     | the two C strings have the same content                  |
| ASSERT_STRNE(str1,str2);     | EXPECT_STRNE(str1,str2);     | the two C strings have different contents                |
| ASSERT_STRCASEEQ(str1,str2); | EXPECT_STRCASEEQ(str1,str2); | the two C strings have the same content, ignoring case   |
| ASSERT_STRCASENE(str1,str2); | EXPECT_STRCASENE(str1,str2); | the two C strings have different contents, ignoring case |

## 9. Linked List 설계
#### Linked List Class
```c++
class LL{
public:
	int data_;
	LL* next_;

	LL() : next_(nullptr) {}
	
	LL* InsertNode(int data){
		LL *NODE = new LL();
		NODE->data_ = data;
		NODE->next_ = nullptr;
		return NODE;
	}
};
```
#### test code
```
#include "gtest/gtest.h"
#include "custom_list.h"
namespace{
class SimpleTest : public ::testing::Test{
	protected:
	void SetUp() override{
		LL1 = new LL();
	}
	// TearDown

	LL* LL1;
};

TEST_F(SimpleTest, InsertNodeTest){
	LL1->InsertNode(1);
	
	EXPECT_TRUE(LL1 != nullptr);
	EXPECT_TRUE(LL1->next_ == nullptr);
	ASSERT_EQ(LL1->data_, 1);
}

}



int main(int argc, char **argv){
	::testing::InitGoogleTest(&argc, argv);
	return RUN_ALL_TESTS();
}

```
#### Makefile
```c++
PHONY: test
GTEST_DIR=googletest/googletest

test:
	g++ -o test test.cpp -isystem ${GTEST_DIR}/include -L${GTEST_DIR}/build -pthread -lgtest
	./test

```
#### Output1
``` 
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from SimpleTest
[ RUN      ] SimpleTest.InsertNodeTest
test.cpp:21: Failure
Expected equality of these values:
  LL1->data_
    Which is: 0
  1
[  FAILED  ] SimpleTest.InsertNodeTest (0 ms)
[----------] 1 test from SimpleTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 0 tests.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] SimpleTest.InsertNodeTest

 1 FAILED TEST

```

- ```LL1->InsertNode(1);``` => ```LL1 = LL1->InsertNode(1);```


```
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from SimpleTest
[ RUN      ] SimpleTest.InsertNodeTest
[       OK ] SimpleTest.InsertNodeTest (0 ms)
[----------] 1 test from SimpleTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test
```

## Reference

- [GoogleTest](https://github.com/google/googletest)
- [Wikipedia Unit_testing](https://en.wikipedia.org/wiki/Unit_testing)
- [GeeksforGeeks](https://www.geeksforgeeks.org/gtest-framework/)
- [googlesource/googletest](https://fuchsia.googlesource.com/third_party/googletest/+/HEAD/googletest/docs/primer.md)