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




### 링크
[참고 링크1](https://d2.naver.com/helloworld/6070967)
[참고 링크2](https://techblog.pet-friends.co.kr/msa%EB%A5%BC-%EA%B0%80%EA%B8%B0-%EC%9C%84%ED%95%9C-%EC%B2%AB%EA%B1%B8%EC%9D%8C-cqrs-%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8-%EB%8F%84%EC%9E%85%EA%B8%B0-510206768a22)