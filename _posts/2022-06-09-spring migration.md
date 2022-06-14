---
layout: post
title:  "springboot version migration"
date:   2022-06-09 18:20:21 +0900
categories: spring
---

# Spring2.4 특징
- Spring Boot 버전을 명시할 때 뒤에 .RELEASE를 작성하지 않아도 됨.
    - ex) 2.4.0.RELEASE → 2.4.0



## Spring 버전 업
> spring boot : 1.5.2 → 2.4.13

> gradle : 3.1 → 6.8.3

> spring-cloud-dependencies: Dalston.SR4 → 2020.0.2

## gradle
compile =>  implementaion

couchbase: com.cocuhbase.client:java-client:2.5.4 → spring-boot-starter-data-couchbase 사용

## Sleuth 변경점

  1. SleuthInterceptor

      HandlerInterceptorAdapter → HandlerInterceptor (Deprecated 되어서 implements 로 변경) 그에 따라 return super.prehandler() → return ture
        tracer.getCurrentSpan() → tracer.nextSpan() (메소드 사라짐)

  2. InterceptorConfig

        WebMvcConfigurerAdapter → WebMvcConfigurer (Deprecated 되어서 implements 로 변경)

  3. percentage → probability 프로퍼티 명 수정


## WebSecurityConfig 추가

    application.yml의 Security basic, path 설정을 WebSecurityConfig로 이동
    그 외 name, password 등의 user정보는 security → spring.security로 이동
    security.enabled 프로퍼티를 통해 path 인증 설정

## ResourceClient
    SimpleClientHttpRequestFactory(deprecated) -> HttpComponentsClientHttpRequestFactory

## Test
    Junit4 → Junit5

    Assert → Assertions

    @Before → @BeforeEach

    @Runwith → @ExtendWith

    @Rule → @ExtendWith({MockitoExtension.class, RestDocumentationExtension.class})

## Spring Cloud 2020.0 버전 업그레이드
    참고 : https://spring.io/projects/spring-cloud 

    2020.0.x aka Ilford -> 2.4.x, 2.5.x (Starting with 2020.0.3)(호환)
    
### bootstrap default사용 중지
- bootstrap.yml 이란?
    - spring-cloud config 파일로 라이프 사이클상 application.yml보다 bootstrap.yml을 먼저 읽어 config server에 정의되어 있는 프로퍼티들을 읽음 
    - re-enable 방법 
        - spring.cloud.bootstrap.enabled=true or spring.config.use-legacy-processing=true 설정
        - spring-cloud-starter-boostrap 사용
    - 변경사항 반영 방법:
        - 현재의 bootstrap.yml 유지
        - application.yml로 properties 이동
            - 현재 bootstrap.yml을 사용하고 있지만 spring cloud config의 역할을 수행하진 않기 때문에 bootstrap.yml를 삭제 후 내부에 있던 application.yml로 properties이동 
            - 과거에 properties들이 application.yml → bootstrap.yml로 분리한 이력이 있음. 그런데 로그포맷 변경(text -> json) 적용 가이드 application.yml로 설정하면 서버 기동에 이슈가 있었음
        - [통합로그].yml을 생성하여 properties이동
            - 현재 bootstrap.yml에서의 properties들은 통합 로그에 사용되는 속성들이니 통합 로그에 관련된 yml파일 생성후 이전 
    
