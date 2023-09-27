---
layout: post
title:  "서킷 브레이커"
date:   2023-09-26 23:20:21 +0900
categories: spring, resilience4j
---

> 회사에 서킷브레이커가 있으면 좋겠다고 판단되어 공부 


### resilience4j
> 마이크로 서비스에서 문제가 되는 트래픽을 차단하여 전체서비스가 느려지거나 중단되는것을 방지

Netflix Hystrix, Resilience4J, Alibaba Sential, Spring Retry와 같은 Circuit Breaker제품들을 사용하기 위해 표준 인터페이스를 제공하는 추상화(또는 Facade) 라이브러리
Hystrix는 업그레이드 지원이 없어, Resilience4J를 사용해야함

써야할 상황
1. API 요청이 실패할 경우(배포, 네트워크 문제 등)..
    - 몇 번 재시도할 것인가?
    - 재시도 간격은 얼마나?
    - 어떤 상황을 호출실패로 볼 것인가?

### 핵심모듈 
1. 서킷브레이커: 요청건수 또는 집계기준으로 서킷 브레이커 제공
    - CLOSE: 정상적
    - OPEN: 차단된 상태
    - HALF_OPEN: 차단된 상태에서 정상적인 상태로 갈 수 있는지 점검
2. Bulkhead: 각 요청을 격리함으로써, 장애가 다른 서비슫에 영향을 미치지 않게함
3. RateLimiter: 요청의 양 조절하여 안정적인 서비스를 제공
4. Retry: 요청이 실패했을 때, 재시도하는 기능
5. TimeLimiter: 응답시간이 지정된 시간을 초과하면 Timeout을 발생
6. Cache: 응답결과를 캐싱

### 주요설정 

```java
resilience4j.circuitbreaker:
  configs:
    default:
      slidingWindowType: COUNT_BASED
      minimumNumberOfCalls: 7                                  # 최소 7번까지는 무조건 CLOSE로 가정하고 호출한다.
      slidingWindowSize: 10                                     # (minimumNumberOfCalls 이후로는) 10개의 요청을 기준으로 판단한다.
      waitDurationInOpenState: 10s                              # OPEN 상태에서 HALF_OPEN으로 가려면 얼마나 기다릴 것인가?

      failureRateThreshold: 40                                  # slidingWindowSize 중 몇 %가 recordException이면 OPEN으로 만들 것인가?

      slowCallDurationThreshold: 3000                           # 몇 ms 동안 요청이 처리되지 않으면 실패로 간주할 것인가?
      slowCallRateThreshold: 60                                 # slidingWindowSize 중 몇 %가 slowCall이면 OPEN으로 만들 것인가?

      permittedNumberOfCallsInHalfOpenState: 5                  # HALF_OPEN 상태에서 5번까지는 CLOSE로 가기위해 호출한다.
      automaticTransitionFromOpenToHalfOpenEnabled: true        # OPEN 상태에서 자동으로 HALF_OPEN으로 갈 것인가?

      eventConsumerBufferSize: 10                               # actuator를 위한 이벤트 버퍼 사이즈

```

### 주요 이벤트

```java
@Override
public void onEntryAddedEvent(EntryAddedEvent<CircuitBreaker> entryAddedEvent) {
    log.info("RegistryEventConsumer.onEntryAddedEvent");

    CircuitBreaker.EventPublisher eventPublisher = entryAddedEvent.getAddedEntry().getEventPublisher();

    eventPublisher.onEvent(event -> log.info("onEvent {}", event));
    eventPublisher.onSuccess(event -> log.info("onSuccess {}", event));
    eventPublisher.onCallNotPermitted(event -> log.info("onCallNotPermitted {}", event));
    eventPublisher.onError(event -> log.info("onError {}", event));
    eventPublisher.onIgnoredError(event -> log.info("onIgnoredError {}", event));

    eventPublisher.onStateTransition(event -> log.info("onStateTransition {}", event));

    eventPublisher.onSlowCallRateExceeded(event -> log.info("onSlowCallRateExceeded {}", event));
    eventPublisher.onFailureRateExceeded(event -> log.info("onFailureRateExceeded {}", event));
}

```

### 어떤 예외를 지정해야할지?
1. 유효성 검사나 NPE 처럼 서킷이 열리는것과 무관한 예외는 RecordException 으로 등록하지말자 
2. Exception이나 RuntimeException 처럼 높은 수준의 예외도 RecordException에 등록하면 안됨
3. 5xx 서버로 오는 것들을 지정 RecordException에 등록

### 서킷브레이커에서 실패했을 때 fallback 메서드 리턴
1. RecordException 발생
2. IgnoreException 발생
3. 서킷이 오픈

```java
@CircuitBreaker(name = SIMPLE_CIRCUIT_BREAKER_CONFIG, fallbackMethod = "fallback")
public String process(String param) throws InterruptedException {
    return callAnotherServer(param);
}

// 지정된 예외를 설정하면 해당 예외로 커스텀할 수 있음.
private String fallback(String param, RecordException ex) {
    log.info("RecordException fallback! your request is " + param);
    return "Recovered: " + ex.toString();
}

    private String fallback(String param, Exception ex) {
        // fallback은 ignoreException이 발생해도 실행된다.
        log.info("fallback! your request is " + param);
        return "Recovered: " + ex.toString();
    }
```



### 링크
[참고 링크1](https://d2.naver.com/helloworld/6070967)
[참고 링크2](https://techblog.pet-friends.co.kr/msa%EB%A5%BC-%EA%B0%80%EA%B8%B0-%EC%9C%84%ED%95%9C-%EC%B2%AB%EA%B1%B8%EC%9D%8C-cqrs-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EB%8F%84%EC%9E%85%EA%B8%B0-510206768a22)
[참고 링크3](https://www.inflearn.com/course/lecture?courseSlug=%EC%9E%A5%EC%95%A0%EC%97%86%EB%8A%94-%EC%84%9C%EB%B9%84%EC%8A%A4-resilience4j-circuitbreaker)