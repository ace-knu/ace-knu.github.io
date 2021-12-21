---
layout: single
title: Intermittent Computing 
---



## Intermittent Computing 이란?
- 에너지 하베스팅(Enger Harvesting)을 통해 적절한 에너지 레벨을 넘어서면 컴퓨팅이 진행되고 이후 에너지가 소모되어 특정 이하의 에너지 레벨이 되면 컴퓨팅을 중지하고 에너지를 다시 충전하게 되는 프로세스를 반복함

  ![01_intermittent_computing](../../assets/img/ic/01_intermittent_computing.png)

- 시스템이 전원 부족으로 중단될 때, 데이터를 보존할 방법으로 백업/복구 루틴을 가지는 코드를 삽입하는 방식으로 개발하여 시스템의 일관성을 유지함


## 현재 연구

- 체크포인트 루틴을 프로그래머가 아닌 컴파일 단계에서 자동 생성 및 삽입 연구

- 시스템의 잔여 에너지량을 고려하여 체크포인트 실행 여부를 결정하는 런타임 체크포인트 활성화 기법 연구


