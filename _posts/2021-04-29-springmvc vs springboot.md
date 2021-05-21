---
layout: post
title:  "SpringMVC vs SpringBoot"
date:   2021-04-29 10:33
categories: spring
tags: [java, spring]
---
### SpringMVC vs SpringBoot

* 스프링 부트
    - 자동으로 Config 구성
    - 라이브러리 의존성 관리 자동
    - 임베디드 서버(톰켓)을 사용하여 단독 실행 가능한 어플리케이션을 구축
    - 의존성관리
    - spring-boot-starter-web
        - spring-web
        - spring-webmvc
        - spring-boot-starter
        - spring-boot-starter-json
        - spring-boot-starter-tomcat
    - AutoConfiguration

* SpringMVC
    - 외장 톰켓 구축
    - 의존성 라이브러리 버전을 함께 정의
        - 버전이 변경될 때마다 모든 버전을 명시해야함
