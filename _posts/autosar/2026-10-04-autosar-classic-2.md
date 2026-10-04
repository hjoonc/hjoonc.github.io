---
title: "[Classic AUTOSAR] SWC, Software Component"
categories: [Automotive Software, AUTOSAR]
tags: [AUTOSAR, Classic AUTOSAR]
---

## SWC의 구조

**SWC**(**Software Component**)는 차량 소프트웨어에서 **특정 기능을 담당하는 소프트웨어 컴포넌트**다.

예를 들어 차량 속도 처리, 목표 토크 계산, 충전 상태 관리 등을 각각 SWC로 구성할 수 있다.

SWC의 구조는 **외부 통신 구조**와 **내부 동작 구조**로 나누어 이해할 수 있다.

## 1. 외부 통신 구조

- **Port**
  - SWC가 외부와 데이터를 교환하거나 서비스를 요청·제공하는 **통신 접점**이다.
  - **P-Port(Provided Port)**
    - 데이터나 서비스를 **제공**하는 Port다.
  - **R-Port(Required Port)**
    - 데이터나 서비스를 **요구**하는 Port다.

- **Interface**
  - Port에서 **교환할 데이터 또는 호출할 Operation을 정의하는 통신 규약**이다.
  - 대표적인 방식으로 **Sender-Receiver**와 **Client-Server**가 있다.
  - **Sender-Receiver(SR)**
    - Sender가 데이터를 제공하고 Receiver가 수신하여 사용하는 방식이다.
    - 주로 **센서 측정값 전달 등에 활용**되며, 차량 속도·온도·목표 토크 등을 전달할 수 있다.
  - **Client-Server(CS)**
    - Client가 **Operation을 호출**하고 Server가 이를 수행하는 방식이다.
    - 주로 **액추에이터의 동작을 요청하는 서비스 등에 활용**할 수 있으며, 필요에 따라 처리 결과를 반환한다.

- **Connector**
  - 서로 호환되는 Interface를 사용하는 **Port들을 연결**한다.
  - 여러 SWC를 연결하여 전체 소프트웨어를 구성한다.

## 2. 내부 동작 구조 — Internal Behavior

**Internal Behavior**는 SWC 내부의 실행 방식과 데이터 접근 등을 정의한다.

- **Runnable Entity(RE): 무엇을 실행하는가**
  - SWC 내부의 동작을 구현하는 **실행 가능한 코드 단위**다.
  - 일반적으로 C 함수로 구현하며, **하나의 SWC 안에 여러 Runnable을 둘 수 있다.**
  - 예: 충전 관리 SWC
    - 충전 상태를 주기적으로 확인하는 Runnable
    - 충전 요청을 처리하는 Runnable

- **Event: 어떤 조건에서 실행하는가**
  - **Runnable의 실행**을 유발하는 **조건 또는 트리거**다.
  - **TimingEvent(TE)**
    - 설정된 주기에 따라 Runnable의 실행을 유발한다.
    - 예: 10ms마다 차량 상태 확인
  - **OperationInvokedEvent(OIE)**
    - Client가 Operation을 호출하면 해당 **Server Runnable의 실행을 유발**한다.
    - 예: 동작 요청 Operation 호출 시 해당 요청을 처리하는 Runnable 실행
  - **ModeSwitchEvent(MSE)**
    - 설정된 모드의 **진입·이탈 또는 전환에 따라 Runnable의 실행을 유발**한다.
    - 예: 특정 운전 모드 진입 시 해당 모드에 필요한 처리 실행
  - 실제 실행 시점은 **OS 스케줄링과 RTE 설정**의 영향을 받는다.

- **Access Point: Interface Data나 Operation에 어떻게 접근하는가**
  - Runnable이 **어떤 Port를 통해 Interface에 정의된 데이터를 읽거나 쓰는지, 어떤 Operation을 호출하는지**를 명세한다.
  - 예: 목표 토크 계산 Runnable이 차량 속도 Port의 Interface에 정의된 속도 데이터를 읽도록 설정한다.
  - **SR 데이터 접근**
    - **Implicit**
      - 입력 데이터를 **Runnable 실행 전에 확보**한다.
      - 한 번의 Runnable 실행 중에는 같은 입력에 대해 **일관된 값**을 사용한다.
      - 출력 데이터는 **Runnable 종료 후 반영**된다.
      - 예: 실행 시작 시 차속이 50이면, 실행 중 외부 값이 51로 갱신되어도 해당 실행에서는 확보된 50을 사용한다.
    - **Explicit**
      - **RTE API를 호출하는 시점에 데이터를 읽거나 쓴다.**
      - 실행 중 같은 데이터를 여러 번 읽으면, **갱신 여부에 따라 서로 다른 값을 얻을 수 있다.**
      - 예: 첫 번째 읽기에서는 차량 속도가 50이고, 갱신 후 두 번째 읽기에서는 51일 수 있다.
  - **CS Operation 호출**
    - **Synchronous Call Point — 동기 호출**
      - Client는 **Server의 처리가 완료되어 호출이 반환될 때까지** 다음 코드로 진행하지 않는다.
      - 예: 계산 결과를 받은 뒤 다음 처리를 수행한다.
      - 호출한 Client가 기다리는 것이며, 다른 Task까지 모두 멈추는 것은 아니다.
    - **Asynchronous Call Point — 비동기 호출**
      - Client는 요청 후 **Server의 완료를 기다리지 않고 실행을 이어갈 수 있다.**
      - 결과는 이후 별도로 확인하거나 수신한다.
      - 예: 작업을 요청한 뒤 다른 처리를 진행하고, 나중에 완료 여부와 결과를 확인한다.

> SWC 내부의 실제 동작은 Runnable로 구현되며, Runnable은 매핑된 OS Task 안에서 실행된다. <br>
> 하나의 Task에서 여러 Runnable을 실행할 수도 있다.
