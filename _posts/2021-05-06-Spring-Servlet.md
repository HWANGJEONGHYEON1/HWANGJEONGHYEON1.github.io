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

## 복습
- 서블릿의 역할
    - 서버 TCP/IP 대기, 소켓연결
    - HTTP 요청 메시지 파싱하여 읽기
    - POST 방식, 및 해당 URL인지
    - ContentType 확인
    - HTTP 바디 내용 파싱
    - HTTP 응답 메시지 생성
    - TCP/IP 응답 전달, 소켓 해제
- HTTP 요청흐름
    - WAS는 req,res 객체를 새로만들어 서블릿 객체 호출
    - 개발자는 req,res 객체에서 요청정보를 활용
    - was는 res 객체에 담겨있는 내용으로 HTTP 응답정보생성

- 서블릿 컨테이너
    - 생명주기 관리
    - 서블릿 객체는 싱글톤으로 관리
    - 고객의 요청일 올 때마다 새롭게 생성하는 것은 비효율적
    - 최초 로딩시점에 서블릿 객체를 만들어 재활용
    - 모든 고객 요청은 동일한 서블릿 인스턴스에 접근
    - 공유 변수 사용시 주의
    - 서블릿 컨테이너 종료 시 함께 종료

- 스레드
    - 서블릿 객체를 호출함
    - 어플리케이션 코드를 하나하나 순차적으로 실행
    - 자바 메인 메서드를 실행하면 메인 이라는 이름의 스레드가 실행
    - 스레드는 한번의 하나의 코드라인만 수행
    - 동시처리가 필요하다면 스레드 생성하여 수행

- 스레드 풀
    - 필요한 스레드를 스레드 개수를 정해논 다음 스레드 풀에 저장
    - 톰켓 최대 200개 가능( 변경가능 )
    - 특징
        - 스레드가 필요할 때 풀에서 꺼내 사용하고 반납
        - 200개의 요청이 들어온 다음 201, 202 요청이오면 대기, 거절 가능
    - 장점
        - 스레드가 미리 생성되어있으므로, CPU 리소스 낭비가 절약되고 응답이 빠름
        - 너무 많은 요청이와도 기존 요청은 안전하게 처리 가능
    



### Ref.
* <https://mangkyu.tistory.com/57>
