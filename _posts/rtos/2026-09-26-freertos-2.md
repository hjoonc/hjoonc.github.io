---
title: "[FreeRTOS] Task 함수 - Suspend, Resume"
categories: [OS, RTOS]
tags: [OS, RTOS, FreeRTOS]
---

## Task Suspend

```c
#include "FreeRTOS.h"
#include "task.h"

void vTaskSuspend( TaskHandle_t pxTaskToSuspend );
```
- 설명
  - **지정한 Task를 Suspended 상태로 전환**한다.<br>
    Suspended 상태의 Task는 **Resume되기 전까지 실행되지 않는다.**

- 매개변수
  - **pxTaskToSuspend**: 중지할 Task의 핸들. **NULL을 전달하면 현재 Task가 스스로 중지된다.**

- 반환값
  - 없음
 

## Task Resume

```c
#include "FreeRTOS.h"
#include "task.h"

void vTaskResume( TaskHandle_t pxTaskToResume );
```

- 설명
  - `vTaskSuspend()`로 중지한 Task를 **Suspended 상태에서 Ready 상태로 전환**한다.

- 매개변수
  - **pxTaskToResume**: 다시 실행 가능하게 만들 Task의 핸들.

- 반환값
  - 없음
