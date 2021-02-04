---
layout: single
title : "AUTOSAR"
---

## 소프트웨어 플랫폼

- 제어계층에서 Application과 HW를 연결해주는 중간 계층임
- 제어기 수량의 증가 및 차량 공간의 제약에 따라 제어기 통합이 필요함
- 제어기 SW의 복잡성이 증가함에 따라 플랫폼 공용화 및 표준화가 필요함

## AUTOSAR 정의

- AUTOmotive Open System ARchitecture의 약자임
- "Cooperate on stadards, compete on implementation" 이라는 기준 아래 형성된 글로벌 파트너쉽임

## AUTOSAR 개발 과정

![개발과정](/assets/img/autosar/개발과정.png)

- AUTOSAR 소프트웨어 개발 순서
  1. Configure System : 시스템 설정 단계로 컴포넌트의 구성/연결 등을 정의함 (Software Component Description을 개발)
  2. Implement Component : Configure System 단계에서 구성한 컴포넌트들에 대한 코드 구현 등을 진행함
  3. Extract ECU-Specific Information : 시스템 구성 정보로부터 특정 제어기 소프트웨어를 구현하기 위한 정보를 추출함
  4. Configure ECU : 제어기 관련 설정을 진행함 (ECU Configuration Description 개발)
  5. Generate Executable : 제어기에서 동작하는 실행 파일을 생성함 (컴파일 및 링크를 통해 실행 이미지를 생성)

## AUTOSAR 플랫폼 구조 

- 차량용 SW Application 개발의 생산성 향상을 위한 표준 플랫폼
  - 계층화된 구조 사용으로 업체별 개발 분담이 가능함 (Application, BSW, MCAL)
- HW와 독립적인 구조로 SW Application 재사용 가능

## AUTOSAR Layered Software Architecture

![layer](/assets/img/autosar/layer.png)

- Application Layer
  - HW 독립적인 어플리케이션 SW를 정의함
  - SWC (SW Component)의 종류는 기능에 따라 Application, Actuator, Sensor 등으로 구별함
  - SWC 간 통신 및 SWC와 BSW 사이의 통신은 RTE를 이용함
- Runtime Environment (RTE)
  - VFB로 모델화된 통신 구조가 실제 로컬 연결이나 네트워크 통신으로 구현된 환경임
  - Application Layer의 SWC간, SWC와 BSW 사이 통신 서비스 제공함
  - ECU 독립적인 Interface Mapping을 SWC에 제공함
  - RTE에 정의된 표준 인터페이스만을 이용하여 SWC를 개발 가능함
- AUTOSAR Interface
  - 사용자 설정에 따라 생성되는 API임
- Standardized AUTOSAR Interface
  - AUTOSAR Interface와 동일한 API 구조를 갖지만 표준에 정의된 Interface임
- Standardized Interface
  - 표준 Interface와 BSW 간 데이터를 주고 받거나 동작을 실행하기 위한 API임
- Basic Software Layers (BSW)
  - Service Layer : 시스템 구동 및 다른 BSW 모듈 제어를 위한 관리 서비스 제공함
  - ECU  Abstraction Layer : MCAL 드라이버들을 상위 계층에 Interface하는 추상화 계층임
  - Micro-Controller Abstraction Layer : MCU 내부 장치를 이용하기 위한 드라이들로 구성함
  - Complex Drivers : AUTOSAR 표준에 정의되지 않은 기능 구현을 위한 계층임