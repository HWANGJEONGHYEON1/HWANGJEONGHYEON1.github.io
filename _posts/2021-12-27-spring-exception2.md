---
layout: post
title:  "api-exception-spring"
date:   2021-12-27 22:30:21 +0900
categories: java, spring, exception
---

> Exception

### HandlerExceptionResolver
- 오류메시지, 형식을 API마다 다르게 보낼 때 사용 
- 상태 코드 변환

### 기존 동작 흐름

~~~
                        preHandle
                        /
WAS -> DispatcherServlet -> handle(handler) -> 핸들러 어뎁터 -> 컨트롤러
               \         \                  <- 전달         (예외발생)
                \        postHandler (얘외 전달안됨)
                 \ 
                 afterCompletion (호출)
~~~

- ExceptionResolver 적용 후
~~~

                       preHandle
                        /
WAS -> DispatcherServlet -> handle(handler) -> 핸들러 어뎁터 -> 컨트롤러
        |      \         \                  <- 전달         (예외발생)
        |       \        ExceptionResolver (예외 해결 시도)
        |        \
 <- View 응답   afterCompletion (호출)

~~~


### ExceptionResolver
> 스프링 MVC는 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다. 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 HandlerExceptionResolver 를 사용하면 된다. 줄여서 ExceptionResolver 라 한다.

- ExceptionResolver 가 ModelAndView 를 반환하는 이유는 마치 try, catch를 하듯이, Exception 을 처리해서 정상 흐름 처럼 변경하는 것이 목적이다. 이름 그대로 Exception 을 Resolver(해결)하는 것이 목적이다.
- 반환 값에 따른 동작 방식
    - HandlerExceptionResolver 의 반환 값에 따른 DispatcherServlet 의 동작 방식은 다음과 같다.

- 스프링 제공
    - ExceptionHandlerExceptionResolver (우선순위 1)
        - @ExceptionHandler를 처리, API 예외처리는 대부분 이 리졸버 사용
    - ResponseStatusExceptionResolver (2)
        - HTTP 상태코드를 지정
        - 커스텀한 예외에 `@ResponseStatus(code = HttpStatus.BAD_REQUEST, reason = "잘못된 요청 오류")`를 처리하면 리졸버에 등록했던 것들을 해준다.
        - response.sendError(statusCode, resolveReason)이 내부에 구현되어있다.
    - DefaultHandlerExceptionResolver (3)
        - 스프링 내부 기본 예외를 처리
        - 스프링이 미리 클라이언트에서 에러가 날만한것들을 셋팅해놔서 알아서 400 상태코드를 내뱉는다(500이 아니라)
            - 예를들어 typemismatch
        
### @ExceptionHandler
- API에 따라 응답의 모양도 다르고 스펙도 다르다.
- API 응답 어려운점
    - HandlerExceptionResolver는 ModelAndView를 리턴해야한다. api 응답은 이것은 필요하지않는다.
    - API 응답을 하기 위해 HttpServletResponse에 add를 해줬다.
    - 특정 컨트롤러에서 발생하는 예외를 별도로 처리하기 어렵다.
    - 동일한 예외 클래스이지만 응답을 좀 다르게 하기 어렵다
- 스프링은 예외처리 문제 해결하기 위해 이 어노테이션을 제공한다. 위 방법들을 해결할 수 있는 방법이다.
- 매커니즘
    - IllegalArgumentsException이 발생
    - ExceptionResolover가 작동 -> 우선순위 제일 높은 ExceptionHandlerExceptionResolver 실행
    - contorller -> handler -> ExceptionHandlerExceptionResolver 호출 -> 컨트롤러에 @ExceptionHandler를 찾음 -> json으로 그대로 리턴
- 지정된 예외 또는 그 하위 자식클래스를 모두 처리할 수 있다.
    - 우선순위
        - 부모 클래스가 있고 자식클래스가 있다면 자식클래스부터 예외가 잡힌다.    


### ControllerAdvice
- 오류와 정삭적인 코드 부분들을 각각 분리하여 코드 작성할 수 있다.
- 대상으로 지정한 컨트롤러에 @ExceptionHandler, @InitBinder 기능을 부여해주는 역할
- 대상을 적용하지 않는다면 적용대상은 모든 컨트롤러, 대상도 적용가능하다.(annotaitions = ~~Controller.class, 또는 패키지)