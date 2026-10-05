---
title: "[FreeRTOS] Timer 함수 - Create, Start, Stop"
categories: [OS, RTOS]
tags: [OS, RTOS, FreeRTOS]
---

## Timer Create

```c
#include "FreeRTOS.h"
#include "timers.h"

TimerHandle_t xTimerCreate( const char *pcTimerName,
                            const TickType_t xTimerPeriod,
                            const UBaseType_t uxAutoReload,
                            void * const pvTimerID,
                            TimerCallbackFunction_t pxCallbackFunction );
```

- 설명
  - 새로운 **소프트웨어 Timer를 생성**하는 함수로,<br>
    Timer의 상태를 관리하는 메모리를 **FreeRTOS Heap에서 자동으로 할당**한다.
  - **Timer를 생성하는 것만으로는 동작하지 않으며**, `xTimerStart()` 등의 함수로 시작해야 한다.
  - Timer를 시작한 후 **설정한 시간이 경과하면**, 등록한 **콜백 함수가 Timer 서비스 Task에서 실행**된다.

- 매개변수
  - **pcTimerName**: **Timer의 이름.** 주로 **디버깅**할 때 사용한다.
  - **xTimerPeriod**: **Timer의 주기.** **Tick 단위**로 지정하며, **0은 사용할 수 없다.**
  - **uxAutoReload**: **Timer의 자동 재시작 여부.** <br>
    **pdTRUE**이면 설정 주기로 반복 동작하는 **자동 재시작 Timer**를, **pdFALSE**이면 **일회성 Timer**를 생성한다.
  - **pvTimerID**: **Timer에 연결할 식별 정보.** <br>
    콜백에서 `pvTimerGetTimerID()`로 읽을 수 있으며, **필요하지 않으면 NULL**을 사용한다.
  - **pxCallbackFunction**: **설정한 시간이 경과했을 때 실행할 콜백 함수의 이름.**

- 반환값
  - **NULL**: **메모리 부족으로 Timer 생성 실패**
  - **NULL이 아닌 값**: **Timer 생성 성공.** 생성된 **Timer의 핸들**을 반환한다.

- 사용 조건
  - `FreeRTOSConfig.h`에서 **configUSE_TIMERS**와 **configSUPPORT_DYNAMIC_ALLOCATION**을 모두 **1**로 설정해야 한다.


## Timer Start

```c
#include "FreeRTOS.h"
#include "timers.h"

BaseType_t xTimerStart( TimerHandle_t xTimer,
                        TickType_t xTicksToWait );
```

- 설명
  - **생성된 Timer를 시작**하는 함수이다.
  - Timer의 시간은 **`xTimerStart()`를 호출한 시점을 기준**으로 계산된다.
  - **이미 동작 중인 Timer에 호출하면**, `xTimerReset()`과 동일하게 **호출 시점부터 시간을 다시 계산**한다.
  - Timer 시작 명령은 **Timer 명령 큐를 통해 Timer 서비스 Task에 전달**된다.
  - **인터럽트 처리 함수(ISR) 내부에서는 `xTimerStartFromISR()`을 사용**한다. <br>
    ISR에서는 Task처럼 대기할 수 없으므로, **ISR 전용 함수**로 Timer를 시작해야 한다.

- 매개변수
  - **xTimer**: **시작하거나 다시 시작할 Timer의 핸들.**
  - **xTicksToWait**: **Timer 명령 큐가 가득 찼을 때, 빈 공간이 생기기를 기다리는 최대 시간.** **Tick 단위**로 지정한다.
    - **0**: 기다리지 않고 즉시 반환한다.
    - **portMAX_DELAY**: `INCLUDE_vTaskSuspend`가 **1**이면 시간 제한 없이 기다린다.
    - **스케줄러 시작 전에 호출하면 이 값은 무시**된다.

- 반환값
  - **pdPASS**: **Timer 시작 명령을 Timer 명령 큐에 전달하는 데 성공.** <br>
    Timer 서비스 Task가 실제로 명령을 처리했다는 의미는 아니다.
  - **pdFAIL**: **Timer 명령 큐가 가득 차서 시작 명령 전달 실패.** <br>
    대기 시간을 지정했다면 해당 시간 동안에도 빈 공간이 생기지 않은 경우이다.

- 사용 조건
  - `FreeRTOSConfig.h`에서 **configUSE_TIMERS**를 **1**로 설정해야 한다.



## Timer Stop

```c
#include "FreeRTOS.h"
#include "timers.h"

BaseType_t xTimerStop( TimerHandle_t xTimer,
                       TickType_t xTicksToWait );
```

- 설명
  - **동작 중인 Timer를 정지**시키는 함수이다.
  - Timer가 정지되면 **해당 Timer의 만료에 따른 콜백 함수가 실행되지 않는다.**
  - Timer 정지 명령은 **Timer 명령 큐를 통해 Timer 서비스 Task에 전달**된다.
  - **인터럽트 처리 함수(ISR) 내부에서는 `xTimerStopFromISR()`을 사용**한다. <br>
    ISR에서는 Task처럼 대기할 수 없으므로, **ISR 전용 함수**로 Timer를 정지해야 한다.

- 매개변수
  - **xTimer**: **정지할 Timer의 핸들.**
  - **xTicksToWait**: **Timer 명령 큐가 가득 찼을 때, 빈 공간이 생기기를 기다리는 최대 시간.** **Tick 단위**로 지정한다.
    - **0**: 기다리지 않고 즉시 반환한다.
    - **portMAX_DELAY**: `INCLUDE_vTaskSuspend`가 **1**이면 시간 제한 없이 기다린다.
    - **스케줄러 시작 전에 호출하면 이 값은 무시**된다.

- 반환값
  - **pdPASS**: **Timer 정지 명령을 Timer 명령 큐에 전달하는 데 성공.** <br>
    Timer 서비스 Task가 실제로 명령을 처리했다는 의미는 아니다.
  - **pdFAIL**: **Timer 명령 큐가 가득 차서 정지 명령 전달 실패.** <br>
    대기 시간을 지정했다면 해당 시간 동안에도 빈 공간이 생기지 않은 경우이다.

- 사용 조건
  - `FreeRTOSConfig.h`에서 **configUSE_TIMERS**를 **1**로 설정해야 한다.
