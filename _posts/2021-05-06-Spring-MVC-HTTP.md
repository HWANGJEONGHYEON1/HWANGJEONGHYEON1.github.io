---
layout: post
title:  "Spring-mvc-http&servlet"
date:   2021-05-06 15:15
categories: spring,http
tags: [spring, http]
---
### HTTP

* 김영한 선생님 강의에서 배운것 - 1

> HTTP
* HTTP 메시지로 모든 것을 전송
* HTML, TEXT, IMAGE, JSON, XML, 거의 모든형태
* 서버간의 데이터를 주고 받음


> Web server
* HTTP 기반으로 동작
* 정적 리소스 제공, 기타 부가기능
* HTML, CSS, JS  
* HTTP 요청 -> HTTP 응답
* APACHE, NGINX

>  Web Application Server
* HTTP 기반으로 동작
* 웹서버 기능 포함 + 정적 리소스 제공
* 프로그램 코드를 실행해서 어플리케이션 로직 수행
* 서블릿, JSP, MVC

> Web Server vs WebApplicationServer
* 웹서버는 정적, WAS는 로직
* 자바는 서블릿 컨테이너 기능을 제공하면 WAS
* WAS는 어플리케이션 코드를 실행하는데 특화

> 서블릿이 없었다면 모든 로직을 개발자가 해야함
* 서버 TCP/IP 연결대기, 소켓 연결
* HTTP 요청  메시지를 파싱
* POST 방식, /save URL 인지
* Content Type 확인
* HTTP 메시지 바디 내용 파싱
* 저장 프로세스 실행
* `비즈니스 로직 실행` -> 서블릿이 탄생하여 비즈니스 로직만 실행하면 됨
* HTTP 응답
* TCP/IP 응답, 소켓 종료

> Servlet
```java
@WebServlet(name = "helloServlet", urlPatterns = "/hello") 
public class HelloServlet extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response){
 //애플리케이션 로직
    }
}
```
* 요청 정보를 편리하게 사용할 수 있는 HttpServletRequest
* 응답정보를 편리하게 사용할 수 있는 HttpServletResponse
* 톰켓처럼 서블릿을 지원하는 Was를 서블릿 컨테이너
* 컨테이너는 객체를 생성 초기화 호출 종료하는 생명주기
* 서블릿 객체는 싱글톤
    - 고객의 요청이 올때마다 객체를 계속 생성하는 것은 비효율적
    - 최초 로딩시점에 서블릿 객체를 만들어 재사용
    - 모든 고객요청은 동일한 서블릿 객체 인스턴스에 접근
    - 공유 변수 사용 주의
    - 서블릿 컨테이너 종료 시 함께 종료
* JSP도 서블릿으로 변환되어서 사용
* 동시 요청을 위한 멀티쓰레드 처리 지원

### 동시 요청 - 멀티쓰레드
> 쓰레드
* 어플리케이션 코드를 하나하나 순차적으로 실행하는 것은 쓰레드
* 자바 메인을 실행하면 main이라는 이름의 쓰레드가 실행
* 쓰레드가 없다면 자바 실행 불가능
* 쓰레드는 한번에 한 코드 라인만 실행
* 동시 처리가 필요하면 쓰레드를 추가 생성

> 요청 마다 쓰레드 생성
- 장점
    - 동시 요청 처리가능
    - 리소스가 허용할 때까지 처리가능
    - 하나의 쓰레드가 지연되어도, 나머지는 동작
- 단점
    - 쓰레드 생성비용 비쌈
        - 응답속도 느려짐(생성하는데 시간이 든다.)
    - 쓰레드는 컨택스트 스위칭 비용이 발생
    - 쓰레드 생성제한이 없다.
        - 고객 요청이 너무 많으면 서버가 죽음.

> 쓰레드 풀 (요청마다 쓰레드 생성의 단점 보완)
- 특징
    - 필요한 스레드를 스레드 풀에 보관 관리
    - 스레드 풀에 생성 가능한 최대치를 관리. 최대200개 기본설정
- 사용 
    - 스레드가 필요하면 이미 생성되어있는 스레드 풀에서 꺼내 사용
    - 종료하면 반납
    - 모두 사용중이라면 요청을 거절하거나 대기하도록 설정
- 장점
    - 스레드가 미리 생성되어있으므로, 스레드를 생성하고 종료하는 비용이 절약, 응답시간 빠름
    - 생성 가능한 스레드의 최대치가 있으므로, 많은 요청이 들어와도 안전하게 처리가능
- 실무
    - WAS의 주요 튜닝 포인트는 최대 스레드 수
        - 낮게 설정한다면?
            - 동시 요청이 많으면, 서버리스소는 여유롭지만, 클라이언트는 응답지연
        - 높다면 ?
            - 동시 요청이 많으면, CPU, 메모리 리소스 임계점 초과로 서버다운
        - 장애 발생 시
            - 클라우드면 서버부터 늘리고 이후 튜닝
        - 적정 숫자
            - 성능테스트
                - 최대 실제 서비스와 유사하게 성능 테스트
                - 툴 : 아파치 ab, 제이미터, nGrinder
- WAS
    - 멀티쓰레드에 대한 부분은 WAS가 처리
    - 개발자가 멀티쓰레드 관련 코드를 신경안써도됨
    - 멀티쓰레드 환경이므로 싱글톤 객체는 주의해서 사용


> HTTP API
* 다양한 시스템 호출
* 주로 JSON 사용
* 데이터만 주고받음, UI 필요하면 클라이언트가 별도 처리
* 서버 TO 서버

> 요청 정보 확인하는 법
* application.properties
    - logging.level.org.apache.coyote.http11=debug
    - 개발 단계에만 적용(성능저하)


### Ref.
* <https://mangkyu.tistory.com/57>
