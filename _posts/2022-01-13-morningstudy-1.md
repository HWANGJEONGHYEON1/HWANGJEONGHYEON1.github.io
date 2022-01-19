---
layout: post
title:  "morningstudy-1"
date:   2022-01-13 08:20:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근)

# 강의 정리(스프링)


### 로그 분석기 만들기
- 모든 public 메서드 호출과 응답 정보를 로그로 출력
- 어플리케이션의 흐름에 영향 x
- 메서드 호출 걸린 시간
- 정상흐름과 예외흐름 구분
- 메서드 호출의 깊이 표현
- http 요청구분
    - http 요청 단위로 특정 Id를 남겨서 어떤 http 요청에서 시작된 것인지 명확히 구분
    - http 시작부터 끝날 때까지 1개의 트랜잭션


```java

    @GetMapping("/v2/request")
    public String request(String itemId) {

        TraceStatus status = null;
        try {
            status = trace.begin("OrderController.request()");
            orderService.orderItem(status.getTraceId(), itemId);
            trace.end(status);
            return "ok";
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;//예외를 꼭 다시 던져주어야 한다.
        }
    }

    //service
    public void orderItem(String itemId) {

        TraceStatus status = null;
        try {
            status = trace.begin("OrderService.orderItem()");
            orderRepository.save(itemId);
            trace.end(status);
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }
    }

    //repo
     public void save(String itemId) {

        TraceStatus status = null;
        try {
            status = trace.begin("OrderRepository.save()");

            //저장 로직
            if (itemId.equals("ex")) {
                throw new IllegalStateException("예외 발생!");
            }
            sleep(1000);

            trace.end(status);
        } catch (Exception e) {
            trace.exception(status, e);
            throw e;
        }

    }


    private void complete(TraceStatus status, Exception e) { 
        Long stopTimeMs = System.currentTimeMillis();
        long resultTimeMs = stopTimeMs - status.getStartTimeMs(); TraceId traceId = status.getTraceId();
        if (e == null) {
            log.info("[{}] {}{} time={}ms", traceId.getId(),
                    addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
        } else {
            log.info("[{}] {}{} time={}ms ex={}", traceId.getId(),
                    addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
        } }

    private static String addSpace(String prefix, int level) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < level; i++) {
            sb.append( (i == level - 1) ? "|" + prefix : "| ");
        }
        return sb.toString();
    }

```

> 2일차

### 필드 동기화
- 모든 요청마다 traceId를 컨트롤러 -> 서비스 -> 레파지토리를 다 넘겨줘야하는 번거러움이 있다.
- 파라미터를 넘기지말고 TraceId를 동기화 할 수 있는 FieldLogTrace

```java
public interface LogTrace {

    TraceStatus begin(String message);

    void end(TraceStatus status);

    void exception(TraceStatus status, Exception e);
}

public class FieldLogTrace implements LogTrace {

    private static final String START_PREFIX = "-->";
    private static final String COMPLETE_PREFIX = "<--";
    private static final String EX_PREFIX = "<X-";

    private TraceId traceIdHolder; // 동시성 이슈 발생, traceId 동기화

```

- 동시성 문제
    - request 시 정상

```
2022-01-17 09:09:32.223  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] OrderController.request()
2022-01-17 09:09:32.223  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] |-->OrderService.orderItem()
2022-01-17 09:09:32.223  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] |   |-->OrderRepository.save()
2022-01-17 09:09:33.227  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] |   |<--OrderRepository.save() time=1004ms
2022-01-17 09:09:33.228  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] |<--OrderService.orderItem() time=1005ms
2022-01-17 09:09:33.228  INFO 18408 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [a4a12386] OrderController.request() time=1005ms
```

    - 비 정상(1초에 두번 호출)
    - 스레드 번호 봤을 시 스레드가 같은 traceId를 쓰고 있다.

```
2022-01-17 09:06:33.240  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] OrderController.request()
2022-01-17 09:06:33.240  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |-->OrderService.orderItem()
2022-01-17 09:06:33.240  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |-->OrderRepository.save()
2022-01-17 09:06:33.916  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |-->OrderController.request()
2022-01-17 09:06:33.917  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |   |-->OrderService.orderItem()
2022-01-17 09:06:33.917  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |   |   |-->OrderRepository.save()
2022-01-17 09:06:34.243  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |<--OrderRepository.save() time=1003ms
2022-01-17 09:06:34.243  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |<--OrderService.orderItem() time=1003ms
2022-01-17 09:06:34.244  INFO 18408 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] OrderController.request() time=1004ms
2022-01-17 09:06:34.920  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |   |   |<--OrderRepository.save() time=1003ms
2022-01-17 09:06:34.921  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |   |<--OrderService.orderItem() time=1004ms
2022-01-17 09:06:34.921  INFO 18408 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [bb55959b] |   |   |<--OrderController.request() time=1005ms
```

> 3일차 

### 동시성 문제
    - FieldLogTrace가 싱글톤으로 등록된 빈, 이 객체의 인스턴스가 하나만 있어서, 여러 요청이 동시에 `FieldLogTrace.traceHolder`를 여러 쓰레드가 동시에 접근해 문제 발생
    - 여러 스레드가 동시에 같은 인스턴스의 필드 값을 변경하면서 발생하는 문제
    - 여러 스레드가 같은 인스턴스의 필드에 접근해야하기 때문에 트래픽이 적은 상황에서는 확율상 잘 나타자지 않고, 트래픽이 늘어날 수록 발생한다.
    - 스프링 빈처럼 싱글톤 객체의 필드를 변경하며 사용할 때 조심해야한다.
    - 참고
        - 동시성 문제는 지역변수에는 발생하지 않는다. 지역변수는 스레드마다 각각 다른 메모리 영익이 할당된다.
        - 동시성 문제가 발생하는 곳은 같은 인스턴스의 필드, 또는 static 같은 공용 필드에 접근할 때 발생한다.
    - 해결방법 : 스레드 로컬
        - 스레드만 접근할 수 있는 특별한 저장소
        - 주의사항
            - 스레드 풀을 사용하는 경우 심각한 문제가된다.(톰켓)
            - 1.사용자 a가 요청
            - 2.was는 풀에서 스레드 조회
            - 스레드 할당
            - 스레드는 사용자의 스레드 로컬에 저장
            - 스레드풀에 반환
            - 스레드 풀에 스레드 로컬이 살아있음.
            - 사용자 b를 조회하는 요청
            - 조회하는 스레드가 하필 스레드 'a'
            - 사용자 a 반환
            - 꼭 ThreadLocal.remove를 사용해야한다.

    - 동시성 문제

```java

private FieldService fieldService = new FieldService();

    @Test
    void field() {
        log.info("mainStart");
        Runnable userA = () -> {
            fieldService.logic("userA");
        };

        Runnable userB = () -> {
            fieldService.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
        sleep(100); // 동시성 문제발생하지 않는다.
        threadB.start();
        sleep(3000);
    }

    public class FieldService {

    private String nameStore;

    public String logic(String name) {
        log.info("저장 name={} -> nameStore={} ", name, nameStore);
        nameStore = name;
        sleep(1000);
        log.info("조회 namestore={}", nameStore);
        return nameStore;
    }

    private void sleep(int millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
09:02:24.510 [thread-A] INFO hello.advanced.trace.threadlocal.code.FieldService - 저장 name=userA -> nameStore=null 
09:02:24.613 [thread-B] INFO hello.advanced.trace.threadlocal.code.FieldService - 저장 name=userB -> nameStore=userA 
09:02:25.520 [thread-A] INFO hello.advanced.trace.threadlocal.code.FieldService - 조회 namestore=userB
09:02:25.619 [thread-B] INFO hello.advanced.trace.threadlocal.code.FieldService - 조회 namestore=userB


```

    - 동시성 문제 해결(ThreadLocal)

```java

    public String logic(String name) {
        log.info("저장 name={} -> nameStore={} ", name, nameStore.get());
        nameStore.set(name);
        sleep(1000);
        log.info("조회 namestore={}", nameStore.get());
        return nameStore.get();
    }

    private ThreadLocalService service = new ThreadLocalService();

    @Test
    void field() {
        log.info("mainStart");
        Runnable userA = () -> {
            service.logic("userA");
        };

        Runnable userB = () -> {
            service.logic("userB");
        };

        Thread threadA = new Thread(userA);
        threadA.setName("thread-A");
        Thread threadB = new Thread(userB);
        threadB.setName("thread-B");

        threadA.start();
        sleep(100); // 동시성 문제발생하지 않는다.
        threadB.start();
        sleep(3000);
    }
```

### Template mehtod pattern
- 템플릿 메서드 패턴은 이름 그대로 템플릿을 사용하는 방식
- 기준이 되는 거대한 틀
- 틀에 변하지 않는 부분을 몰아두고 변하는 부분은 별도 호출해해서 사용