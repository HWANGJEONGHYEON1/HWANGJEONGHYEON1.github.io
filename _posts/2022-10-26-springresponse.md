---
layout: post
title:  "스프링 Response 동작과정(how to response to json)"
date:   2022-10-24 18:20:21 +0900
categories: kafka
---

> 스프링이 어떻게 JSON으로 리턴하는지 디버깅 시도 진짜 ObjectMapper를 사용하는가?

## 흐름
1. 컨트롤러 수행 
![controller](https://raw.githubusercontent.com/HWANGJEONGHYEON1/HWANGJEONGHYEON1.github.io/master/images/correct_handler.png)
2. DispatcheServlet이 적합한 Adapter 연결 (AbstractHandlerMethodAdapter)
3. 적합하 리턴을 시켜줄 리턴 핸들러 호출 (HandlerMethodReturnValueHandler)
![invokeHandle](https://raw.githubusercontent.com/HWANGJEONGHYEON1/HWANGJEONGHYEON1.github.io/master/images/invokeAndHandle.png)
4. 리턴 준비 => MappingJackson2HttpMessageConverter가 할당된걸 볼 수 있다.
![genericHttpMessageConverter](https://raw.githubusercontent.com/HWANGJEONGHYEON1/HWANGJEONGHYEON1.github.io/master/images/genericHttpMessageConverter.png)
    - 메시지 컨버터 목록
    ![genericHttpMessageConverter](https://raw.githubusercontent.com/HWANGJEONGHYEON1/HWANGJEONGHYEON1.github.io/master/images/messageConverter.png)
5. AbstractGenericHttpMessageConverter에서 objectmapper를 사용. 
![genericHttpMessageConverter](https://raw.githubusercontent.com/HWANGJEONGHYEON1/HWANGJEONGHYEON1.github.io/master/images/objectmapper.png)

## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D