---
layout: post
title:  "springboot version migration"
date:   2022-06-09 18:20:21 +0900
categories: spring
---

# Spring 버전 업
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
