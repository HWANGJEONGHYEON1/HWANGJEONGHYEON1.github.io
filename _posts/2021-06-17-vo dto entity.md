---
layout: post
title:  "VO DTO ENTITY 차이"
date:   2021-06-17 01:50
categories: vo, dto, entity
tags: [vo, dto, entity]
---

### vo, dto, entity 차이점을 알아보자.

> 언제 쓸것인지 잘 구분한다면 기준점을 지을 수 있다.

### VO
- value obejct
- 값 객체
- equals() & hashcode() 오버라이딩
- 내부에 선언된 필드 모든 값들이 vo 객체마다 다 똑같아야함
- 객체의 불변성을 보장한다.


### DTO
- data transfer object
- 주로 비동기 처리할 때 사용
- 스프링 부트에서 Jackson 라이브러리 이용하여 json 타입으로 변환해주는데 이 때 사용한다.
- 데이터 가공처리

### Entity
- DB 테이블 내에 존재하는 컬럼만을 필드로 가지는 클래스
- 상속을 받거나 구현체여서는 안된다,
- 테이블 내에 존재하지 않는 컬럼이 있으면 안된다.
- 주로 JPA



 ### Ref.
* <https://webdevtechblog.com/entity-vo-dto-666bc72614bb>
