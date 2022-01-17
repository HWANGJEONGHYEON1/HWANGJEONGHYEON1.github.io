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

- 동시성 문제
    - FieldLogTrace가 싱글톤으로 등록된 빈, 이 객체의 인스턴스가 하나만 있어서, 여러 요청이 동시에 `FieldLogTrace.traceHolder`를 여러 쓰레드가 동시에 접근해 문제 발생