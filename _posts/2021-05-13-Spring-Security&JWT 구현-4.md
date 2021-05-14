---
layout: post
title:  "Spring-Security 구현 - 4"
date:   2021-05-09 01:15
categories: spring,security
tags: [security, jwt]
---
### Spring-Security 구현 - 4

* 프로젝트에 Spring Security, JWT 구현한다.

* Jwt 토큰 저장어디다가 하는것이 좋나
    - Cookie 와 Local Storage
    - 누가 데이터를 필요로 하는 것인가에 판단
    - Cookie
        - 서버 측 읽기용
        - 유효기간
        - Cookie는 HttpOnly 옵션과 Secure 옵션을 통해서 XSS공격을 방어할 수 있다.
        - 즉 자바스크립트가 쿠키를 조작하는것을 막아버리기 때문에 사용자의 토큰 값이 변경될 염려를 하지 않아도 된다는 장점이 있다.
        - csrf 공격에 허용 하지만 Jwt는 토큰 기반이므로 괜찮을 것 같다.
    - Local Storage 
        - 클라이언트 측 읽기용
        - 데이터 영구저장

* Http Request 처리 과정 (DispatcherServlet)
    - 다시 익히기
    - `FrontController` 디자인 패턴에 따라 Spring Web MVC에서는 서버로 오는 모든 네트워크 요청을 분류하는 역할
    - `HandlerMapping`
        - HandlerMapping을 상속하는 RequestMappingHandlerMapping을 통해 설정
        - `@RequestMapping`을 확인하고 dispatcherServlet에 등록
    - `ViewResolver`
        - `UrlBasedViewResolver`가 보통 적용
        - 반환 시 redirect: or forward: 를 포함





### Ref.
* <https://stackoverflow.com/questions/3220660/local-storage-vs-cookies>
* <https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-lazy-initialization>

        
