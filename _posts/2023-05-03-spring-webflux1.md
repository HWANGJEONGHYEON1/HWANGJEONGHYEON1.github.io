---
layout: post
title:  "webflux - 1"
date:   2023-05-03 23:20:21 +0900
categories: webflux
---

> Spring webflux

## 장점, 단점
- 장점
    - 효율적인 리소스 사용
    - 요청이 순간적으로 늘어나도 유연하게 커버
    - 동시성을 극한으로 응답속도 단축
- 단점
    - 비동기로직 처리에 고민
    - 디버깅, 에러핸들링 어려움


## Reactive
- 동기와 비동기, Blocking과 Non-Blocking의 차이점은?
- Reactive system의 필수요소
- Reactive streams의 구조는?
- Java nio는 어떻게 동작하는가, java io의 차이는?
- Reactor pattern을 사용해서 어떤일을 하는가?
- Reactive는 무엇인가?


### 비동기란?
- Caller, Callee
    - Caller : 호출하는 함수
    - Callee : 호출 당하는 함수
- 동기
    - Caller는 Callee의 결과에 관심이 있다.
    - Caller는 결과를 이용해 action을  취한다.
- 비동기
    - Caller는 Callee의 결과에 관심이 없다.
    - Callee는 결과를 이용하여 callback을 수행한다.
- 함수형 인터페이스
    - `호출한 쓰레드에서 실행된다` 
- Blocking
    - Caller는 Callee가 완료되기 전까지 아무것도 할 수 없다
    - 제어권을 Callee가 가지고 있다.
    - caller와 다른 별도의 스레드가 필요하지 않다.
- Non-blocking
    - Caller는 본인의 일을 할 수 있다.
    - 제어권을 Caller가 가지고 있다.
    - caller와 다른 별도의 스레드가 필요하다. (다른 일을 해야하기 때문에)

||동기|비동기|
|Blocking| caller는 결과에 관심있고 아무것도 할 수 없다.| Caller는 아무것도 할 수 없고 결과는 Callee가 처리한다.|
|NonBloking|Caller는 결과에 관심있고 얻은 후에 직접 할 수 있다.| Caller는 자기할 일을 할 수 있고, 결과는 Callee가 처리한다.|


### CompletableFuture
- Future
    - 비동기적인 작업 수행
    - 해당 작업이 완료되면 결과를 반환하는 인터페이스
- CompletionStage
    - 비동기적인 작업 수행
    - 해당 작업이 완료되면 결과를 처리하거나 다른 CompleStage를 연결하는 인터페이스
- ExecutorService
    - 스레드 풀을 이용하여 비동기적으로 작업을 실행하고 관리
    - 별도의 스레드를 생성하지 관리하지 않아도 되므로, 코드를 간결하게 유지가능
    - 스레드 풀을 이용하여 자원을 효율적으로 관리
