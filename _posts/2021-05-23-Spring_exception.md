---
layout: post
title:  "스프링부트 예외처리하기"
date:   2021-05-23 23:10
categories: spring, exception, controllerAdvice
tags: [java, exception]
---

### 스프링 부트 예외처리

* 계속 업데이트 예정

#### @ControllerAdvice
    - `@ExceptionHandler, @ModelAttribute, @InitBinder`가 적용된 메서드들을 AOP를 적용하여 컨트롤러 단에 적용하기 위한 어노테이션이다.
    - @Controller, 전역에서 발생할 수 있는 예외를 잡아 처리해주는 annotaion
    - 뷰 리졸버를 통해 예외 처리페이지를 리다이렉트 처리

#### @RestControllerAdvice
    - @ResponseBody + @ControllerAdvice
    - API 서버의 에러 응답으로 객체를 리턴
    - 컨트롤러어드바이스와 역할은 동일 하지만 응답을 뷰 가아니라 json으로 응답
    
#### 코드

```java
@ControllerAdvice
@Slf4j
public class CommonExceptionHandler {

    @ExceptionHandler
    public String errorHandler(Exception e, Model model) {

        log.error("Exception ... " + e);

        model.addAttribute("exception", e);

        return "/error/error_page";
    }
}

```
