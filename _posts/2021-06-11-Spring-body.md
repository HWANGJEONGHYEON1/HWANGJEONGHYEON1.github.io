---
layout: post
title:  "스프링 요청,응답 받는법"
date:   2021-06-13 19:45
categories: spring, request, response
tags: [spring, request, response]
---

### @ResponseBody, @ResquestBody

> 아무 생각없이 쓰다가 개인프로젝트를 진행하면서 값 전달이 안되어 찾아보고 정리

### @ResquestBody

- 클라이언트가 전송하는 `json` 형태의 `HTTP body` 내용을 `Java Object`로 변환 시켜주는 역할
- 바디가 존재하지 않는 Get Method를 사용한다면 에러 발생
- MappingJackson2HttpMessageConverter를 통해 객체 변환

### @ResponseBody
- 자바 객체를 HTTP 응답 객체 바디로 전달
- HTTP 요청의 바디 내용으로 매핑하는 역할


> 이름 그대로만 생각해도 응답의 바디, 요청의 바디이다. 




