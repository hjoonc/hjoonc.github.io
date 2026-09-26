---
title: "[FreeRTOS] Task 함수 - Create, Delay, DelayUntil"
categories: [OS, RTOS]
tags: [OS, RTOS, FreeRTOS]
---

## Task Create

```c
#include "FreeRTOS.h"
#include "task.h"

BaseType_t xTaskCreate( TaskFunction_t pvTaskCode,
                        const char * const pcName,
                        unsigned short usStackDepth,
                        void *pvParameters,
                        UBaseType_t uxPriority,
                        TaskHandle_t *pxCreatedTask );
```
- 설명
  - 새로운 Task를 생성하는 함수로, Task에 필요한 TCB와 Stack 메모리를 FreeRTOS Heap에서 자동으로 할당한다.
  - 생성된 Task는 Ready 상태로 등록되며, 실행 가능한 다른 Task와 우선순위에 따라 실행된다.

- 매개변수
  - **pvTaskCode**: **Task가 실행할 함수의 이름.** 일반적으로 **함수 내부**에 **무한 루프를 작성**한다.
  - **pcName**: **Task의 이름.** 주로 **디버깅**할 때 사용한다.
  - **usStackDepth**: **Task에 할당할 Stack의 크기.** 바이트가 아닌 **워드(word) 단위**로 지정한다.
  - **pvParameters**: **Task 함수에 전달할 매개변수.** **전달할 값이 없으면 NULL**을 사용한다.
  - **uxPriority**: **Task의 우선순위.** **0이 가장 낮으며, 값이 클수록 우선순위가 높다.**
  - **pxCreatedTask**: **생성된 Task의 핸들을 저장할 변수.** 이후 Task를 참조할 필요가 없으면 NULL을 사용한다.
 
- 반환값
  - **pdPASS**: **Task 생성 성공**
  - **errCOULD_NOT_ALLOCATE_REQUIRED_MEMORY**: **메모리 부족으로 Task 생성 실패**

## Task Delay

```c
#include "FreeRTOS.h"
#include "task.h"

void vTaskDelay( TickType_t xTicksToDelay );
```
- 설명
  - **현재 Task를 지정한 Tick 수만큼 Blocked 상태로 전환**한다. **지연 시간이 지나면 다시 Ready 상태**가 된다.
 
- 매개변수
  - xTicksToDelay: 현재 Task가 Blocked 상태로 기다릴 Tick 수를 지정한다.

- 반환값
  - 없음

## Task DelayUntil

```c
#include "FreeRTOS.h"
#include "task.h"

void vTaskDelayUntil( TickType_t *pxPreviousWakeTime,
                      TickType_t xTimeIncrement );
```
- 설명
  - **지정한 시각까지 현재 Task를 Blocked 상태로 전환**한다.<br>
    **vTaskDelay()가 호출한 시점부터 지정한 Tick 수만큼 기다리는 것과 달리**,<br>
    **vTaskDelayUntil()은 이전 실행 시각을 기준으로 다음 실행 시각을 계산**한다.

- 매개변수
  - **pxPreviousWakeTime**: **이전 실행 시각을 저장한 변수의 주소**.<br>
    **xTaskGetTickCount()로 초기화**하며, **이후에는 함수가 다음 주기를 계산할 수 있도록 값을 자동으로 갱신**
  - **xTimeIncrement**: 이전 실행 시각을 기준으로, **현재 Task가 다음에 깨어날 시각을 정하는 Tick 간격**이다.
 
- 반환값
  - 없음

## Task Delay vs Task DelayUntil

![DLDLUNTIL](/assets/img/posts/rtos/pic1-1-dldluntil.png)
- **vTaskDelay()는 호출한 시점부터 지정한 시간만큼 기다리기 때문에**, 작업에 걸린 시간에 따라 **다음 실행 시점이 달라진다.** <br>
- 반면 **vTaskDelayUntil()은 이전 실행 시각을 기준으로 다음 실행 시각을 계산**하기 때문에, 작업 시간이 조금씩 달라도 **일정한 실행 주기를 유지**할 수 있다.


## 참고사항
- Tick의 주기

  - **Tick 하나의 시간**은 **FreeRTOSConfig.h**의 **configTICK_RATE_HZ** 설정에 따라 결정된다.<br>
    예를 들어 **configTICK_RATE_HZ가 1000**이면 **1초에 Tick이 1000번 발생**하므로, **1 Tick = 1ms**이다.

- DelayUntil의 pxPreviousWakeTime

  - Task를 Suspend했다가 **Resume**할 때는,<br>
    **재개 시점부터 주기를 다시 시작하려면 xTaskGetTickCount()로 기준 시각을 갱신**한다.
