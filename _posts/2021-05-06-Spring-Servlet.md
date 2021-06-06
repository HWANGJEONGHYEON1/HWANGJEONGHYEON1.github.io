---
layout: post
title:  "Spring-servlet"
date:   2021-05-06 18:15
categories: spring,http
tags: [spring, http]
---
### Servlet

* 배운 내용

> HttpServletRequest
* 서블릿은 HTTP 요청 메시지를 편리하게 사용할 수 있도록 해줌
* 요청 메시지를 개발자 대신에 HTTP 요청메시지를 파싱해 그 결과를 담아줌

> 중요
- HttpServletRequest, HttpServletResponse를 사용할 때 중요한점은 HTTP 요청, 응답 메시지를 편하게 도와주는 객체
- 깊이 이해하려면 HTTP 스펙이 제공하는 요청, 응답 메시지 자체를 이해하면 좋다.

> HTTP 요청 데이터
* Get - 쿼리 파라미터
    - /url?username=aa&age=10 
    - 메시지 바디 없이 쿼리 파라미터 형식으로 전달
    - 검색, 필터 등
* POST
    - content-type: application/x-www-form-urlencoded
    - 메시지 바디에 쿼리파라미터 형식으로 전달
    - 회원가입, 상품주문, html form 등
* HTTP message body
    - HTTP API 주로사용, JSON, XML, TEXT
    - 주로 JSON
    - POST PUT PATCH

    


### Ref.
* <https://mangkyu.tistory.com/57>
