---
layout: single
title: Software with NVRAM
---



## NVRAM이란?

- NVRAM (Non-Volatile RAM, 비휘발성 램)은 전원이 공급되지 않아도 기억내용을 유지할 수 있는 RAM 메모리

- 대표적인 NVRAM으로는 FRAM (Ferroelectric RAM, 강유전체 램)이 있으며 메모리 시장의 대부분을 차지하고 있는 EEPROM, 플래시 메모리에 비해 낮은 전력 소모, 빠른 속도, 높은 쓰기/지우기 횟수 등의 여러 장점을 가지고 있음

  ![fram_spec](/assets/img/nvram/fram_spec.png)

- EEPROM이나 플래시 메모리만큼의 집적도나 낮은 가격을 실현하지 못하고 있지만 스마트 카드 등의 몇몇 특수한 분야에서는 유용하게 활용되고 있음





## NVRAM 기반 시스템 소프트웨어 연구

- NVRAM 추상화 계층 설계 및 드라이버 개발

  ![nvram_abstraction_layer](/assets/img/nvram/nvram_abstraction_layer.png)

  - 시스템 내에 포함되는 NVRAM을 효율적으로 사용하기 위해 NVRAM을 추상화하는 계층이 필요함
  - 칩 내/외부에 존재하는 NVRAM을 단계적으로 추상화하는 메모리 서비스 계층과 메모리 하드웨어 추상화 계층을 설계함
  - 각 계층에서의 동작을 위한 인터페이스를 설계하고 내/외부 NVRAM에 접근하기 위한 드라이버를 개발함



- NVRAM 기반 임베디드 메모리 계층 설계

  ![nvram_memory_hierarchy](/assets/img/nvram/nvram_memory_hierarchy.png)

  - NVRAM를 다양하게 활용하기 위해 NVRAM 기반 시스템의 메모리 계층에 대한 재설계가 필요함
  - NVRAM을 효과적으로 사용하기 위해 용도에 따른 영역을 설계함
  - 각 영역에 접근하여 동작을 수행할 수 있는 인터페이스를 설계하고 드라이버를 개발함
