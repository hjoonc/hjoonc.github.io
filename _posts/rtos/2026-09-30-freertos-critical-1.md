---
title: "[FreeRTOS] Critical 함수 - Enter, Exit"
categories: [OS, RTOS]
tags: [OS, RTOS, FreeRTOS]
---

## Critical Enter

```c
#include "FreeRTOS.h"
#include "task.h"

void taskENTER_CRITICAL( void );
```

- 설명
  - **Critical Section(임계 구역)에 진입**하는 매크로이다.
  - 임계 구역은 **공유 데이터에 접근하는 코드 등을 보호하기 위해 다른 실행 흐름의 개입을 제한하는 구간**이다.
  - **인터럽트를 전체 또는 특정 우선순위 범위에서 비활성화**하여 임계 구역을 구현한다.
  - 작업이 끝나면 **`taskEXIT_CRITICAL()`을 호출하여 빠져나와야 한다.**

- 매개변수
  - 없음.

- 반환값
  - 없음.


## Critical Exit

```c
#include "FreeRTOS.h"
#include "task.h"

void taskEXIT_CRITICAL( void );
```

- 설명
  - **Critical Section(임계 구역)에서 빠져나오는 매크로**이다.
  - **`taskENTER_CRITICAL()`과 짝을 이루어 사용**한다.
  - 가장 바깥쪽 임계 구역까지 종료하면 **임계 구역에 의한 인터럽트 차단이 해제**된다.

- 매개변수
  - 없음.

- 반환값
  - 없음.


## 공통 동작 및 주의 사항

- 인터럽트 차단 범위
  - **포트는 FreeRTOS를 특정 MCU에서 동작하도록 구현한 코드**이며, 포트에 따라 인터럽트 차단 방식이 다르다.
  - **전체 차단 방식의 포트**:
    - configMAX_SYSCALL_INTERRUPT_PRIORITY를 사용하지 않으며, **인터럽트를 전체적으로 비활성화**한다.
  - **우선순위 기준 차단 방식의 포트**:
    - FreeRTOSConfig.h에서 설정한 **configMAX_SYSCALL_INTERRUPT_PRIORITY 값을 차단 기준으로 사용**한다.
    - 기준과 같거나 낮은 우선순위의 인터럽트는 차단된다.
    - 기준보다 높은 우선순위의 인터럽트는 계속 실행될 수 있으므로, 해당 ISR과 공유하는 데이터는 별도 보호가 필요하다.
    - 포트에 따라 설정 이름이 configMAX_API_CALL_INTERRUPT_PRIORITY일 수 있다.

- Task 전환
  - 관련 인터럽트가 차단되어 **선점형 Task 전환이 발생하지 않으며**, 현재 Task가 **Running 상태를 유지**한다.
  - 내부에서는 **Task를 Blocked 상태로 만들거나, `yield`로 실행을 양보하면 안 된다.**

- 중첩 호출
  - **임계 구역은 중첩해서 진입할 수 있다.**
  - 진입한 횟수만큼 **`taskEXIT_CRITICAL()`을 호출해야 완전히 빠져나온다.**
  - **가장 바깥쪽 임계 구역이 종료될 때까지 인터럽트 차단이 유지**된다.

  ```c
  taskENTER_CRITICAL();      // 0 → 1: 진입
  taskENTER_CRITICAL();      // 1 → 2: 추가 진입

  taskEXIT_CRITICAL();       // 2 → 1: 안쪽 구역 종료, 바깥쪽 유지
  taskEXIT_CRITICAL();       // 1 → 0: 바깥쪽 구역까지 종료
  ```

- 주의 사항
  - 인터럽트 응답 지연을 줄이도록 **임계 구역은 최대한 짧게 유지**해야 한다.
  - 진입·종료 호출은 **반드시 짝을 맞추며**, `return` 등으로 종료 호출을 건너뛰면 안 된다.
  - **ISR 내부에서는 Task용 매크로 대신 아래 ISR용 매크로를 사용**한다.
    - 진입: `taskENTER_CRITICAL_FROM_ISR()`
    - 종료: `taskEXIT_CRITICAL_FROM_ISR()`
  - 내부에서는 **FreeRTOS API 함수를 호출하면 안 된다.** <br>
    단, 중첩 진입·종료를 위한 매크로 호출은 가능하다.
  - 인터럽트 차단 없이 **Task 스케줄링만 일시 중지**하려면 `vTaskSuspendAll()`을 참고한다. <br>
    이때 **ISR은 계속 실행되므로 ISR의 공유 데이터 접근까지 막지는 못한다.**
