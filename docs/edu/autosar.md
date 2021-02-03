---
layout: single
title : "AUTOSAR"
---


<div style=" font: italic bold 1.5em/1em Georgia, serif ;">
<li>
제어계층에서 어플리케이션과 HW를 연결해주는 중간 계층임
제어기 수량의 증가 및 차량 공간의 제약에 따라 제어기 통합이 필요함
제어기 SW의 복잡성이 증가함에 따라 플랫폼 공용화 및 표준화가 필요함
</li>


AUTOSAR 정의

- AUTOmotive Open System ARchitecture의 약자임
- "Cooperate on stadards, compete on implementation" 이라는 기준 아래 형성된 글로벌 파트너쉽임



AUTOSAR 개발 과정

- AUTOSAR 소프트웨어 개발 순서
  1. Configure System : 시스템 설정 단계로 컴포넌트의 구성/연결 등을 정의함 (Software Component Description을 개발)
  2. Implement Component : Configure System 단계에서 구성한 컴포넌트들에 대한 코드 구현 등을 진행함
  3. Extract ECU-Specific Information : 시스템 구성 정보로부터 특정 제어기 소프트웨어를 구현하기 위한 정보를 추출함
  4. Configure ECU : 제어기 관련 설정을 진행함 (ECU Configuration Description 개발)
  5. Generate Executable : 제어기에서 동작하는 실행 파일을 생성함 (컴파일 및 링크를 통해 실행 이미지를 생성)



