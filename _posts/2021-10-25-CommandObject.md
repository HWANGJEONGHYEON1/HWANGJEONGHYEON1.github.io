---
layout: post
title: "@ModelAttribute 객체 동작과정"
date:  2021-10-25 16:20
categories: spring
tags: [spring]
---

## 개발을 하면서 어떻게 DTO 객체가 자동으로 바인딩 될 수 있지?

### 커멘드 객체
- HttpServletRequest를 통해 들어온 요청 파라미터들이 setter를 이용하여 객체에 정의되어있는 속성에 바인딩 되는 객체
- HttpServletRequest로 받아오는 요청 파라미터의 key 값과 동일한 이름의 속성과 setter를 가지고 있어야한다.

### @ModelAttribute
- 커멘드 객체와 같이 요청 파라미터들을 객체 프로퍼티에 바인딩
- @ModelAttribute를 생략해도 커맨드 객체를 이용해 바인딩이 된다.
- @RequestParam도 생략가능
- 스프링이 내부적으로 String, int 등은 생략됬다고 본다(@RequestParam), 복잡한 객체들은 @ModelAttribute가 생략됬다고 판단한다,.
- model에 객체를 담아준다.
- @RequestParam과는 달리 검증 작업을 내부적으로 진행한다.
  - int type일 때 문자가 들어온다면 클라이언트에 Http 400 에러를 응답한다.
- @ModelAttribute는 400 에러가 발생하지 않고 BindingException 타입의 객체가 담겨 컨트롤러로 전달


### 참고
- `@ModelAttribute`는 Http 요청 파라미터(URL 쿼리 스트링, POST form)을 다룰 때 사용
  - Http 요청 파라미터르 처리는 각각의 필드 단위로 세밀하게 적용, 그래서 특정 필드 타입에 맞지 않는 오류가 발생해도 나머지 필드는 정상처리됨
- `@RequestBody`는 Http의 바디 데이터를 객체로 변환할 때 사용(JSON)
  - HttpMessageConverter는 JSON 데이터를 객체로 변환하지 못하면 이후 단계 자체가 진행하지 않고 예외가 발생한다 -> 컨트롤러 호출불가
  
### 링크
- https://velog.io/@gillog/Spring-Command-Obejct
- https://webdevtechblog.com/modelattribute-%EC%99%80-%EC%BB%A4%EB%A7%A8%EB%93%9C-%EA%B0%9D%EC%B2%B4-command-object-42c14f268324
