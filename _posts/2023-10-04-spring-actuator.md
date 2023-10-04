---
layout: post
title:  "서킷 브레이커"
date:   2023-10-04 23:20:21 +0900
categories: spring, actuator
---

> Spring actuator

### 기본제공 endpoints
> endpoint 는 enable/disable (활성화 여부)과 expose(노출 여부) 라는 2가지 설정을 할 수 있으며 2가지 모두 켜진 상태여야 외부로 노출 되어야한다.

- beans: 등록된 bean 목록 제공
- caches: cache 사용중이라면 cache 관련 정보 제공
- conditions: spring auto configuration 에 의해 bean으로 등록된것과 그렇지 않은 것의 상세 이유를 제공
- health: application이 구동중인지, application과 연동되는 다른 서비스(DB, message queue)가 구동중인지 여부 제공
- info: application 정보
- metrics: 로거 설정확인 실시간 로그 레벨 변경 제공
- quartz

### 보안문제
actuator 를 통해 application의 다양한 정보를 확인할 수 있고, 특정 endpoint 에서는 실시간 변경도 가능하게 해준다. (ex, thread dump) 따라서 보안상 문제가 있을 수 있으므로 spring security 혹은 이와 유사한 방법으로 보안 위험을 해결함.  가장 쉬운 방법은 spring security 를 통해 /actuator url 에 대해 http basic auth 을 적용해서 id, pw 가 맞아야만 pass 되도록 권장

### cache & cors
endpoint 마다 캐시값을 설정해주고 `management.endpoint.<endpoint명>.cache.time-to-live` 에 값을 적어주면 된다.

```
management:
  endpoints:
    web:
      cors:
        allowed-origins: http://domain.com
        allowed-methods: GET
```

### health
- show-components
    - show-components: ALWAYS 
        - components 필드에 기본 상태 추가(ping, diskSpace)
- show-details 
    - show-details: ALWAYS
        - 각 컴포넌트의 디테이한 정보 추가
- 기본 제공 정보
    - cassandra, couchbase, db, dispace, elk ...
    - 각 설정관련 의존성 추가 후 기동하면 auto configuration을 통해 설정 정보가 보임

### metrics
> 회사에서 운영/모니터링시 주로 사용하는게 cpu, mem, disk usage, thread count, cache 용량 등인데 이런 정보는 대부분 metrics endpoint 에서 제공된다.

- https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics.supported
- 외부 시스템과 연동을 설정파일에 적어주면 spring에서 자체적으로 제공함 예를들어, elk, datadog, prometheus ...

```
{
  "names": [
    "hikaricp.connections",
    "hikaricp.connections.acquire",
    "hikaricp.connections.active",
    "hikaricp.connections.creation",
    "hikaricp.connections.idle",
    "hikaricp.connections.max",
    "hikaricp.connections.min",
    "hikaricp.connections.pending",
    "hikaricp.connections.timeout",
    "hikaricp.connections.usage",
    "http.server.requests",
    "jvm.buffer.count",
    "jvm.buffer.memory.used",
    "jvm.buffer.total.capacity"
    ...
    ]
}
```


### counter
> 캐시 hit 또는 http request 누적 횟수같은 것들을 사용할 때 가능

1. Counter.builder() -> 잘 활용안함
2. @Counted 사용
    - AOP로 동작하여 해당 요청 metrics 정보를 조회할 수 있음

### timer
> 특정 메서드의 수행 시간을 timer metric 에 저장할 수 있음

- CountedAspect 빈 등록 후
- @Timed 사용


### 링크
[참고 링크1](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)
[참고 링크2](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.implementing-custom)