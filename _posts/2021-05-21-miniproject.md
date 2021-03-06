---
layout: post
title:  "JPA 맛보기"
date:   2021-05-21 14:40
categories: spring, template, thymeleaf
tags: [spring, jpa]
---

### JPA - 맛보기

* 코드 생략

#### 앤티티 설계시 주의점
- 모든 연관관계는 지연로딩으로 설정해야한다.
    - 즉시로딩(EAGER)은 예측이 어렵고, 어떤 SQL이 실행이 되는지 파악하기 어렵다. 특히 JPQL을 실행할 때 N + 1 문제가 자주 발생한다.
    - 실무에서 모든 연관관계는 (LAZY)로 설정해야한다.
    - 연관된 엔티티를 함꼐 db에서 조회하면, fetch join 또는 엔티티 그래프 기능을 사용한다.
    - @XToOne(OneToOne, ManyToOne)
- 컬렉션은 필드에서 초기화하는 것이 안전하다.
    - `Null` 문제에서 안전하다.
    - 하이버네이트는 엔티티를 영속화 할 때, 컬렉션을 감싸서 하이버네이트가 제공하는 내장 컬렉션으로 변경한다.
- 하이버넹니트 기존 구현 : 엔티티의 필드명을 그대로 테이블 명으로 사용
    - SpringPhysicalNamingStrategy
    - 카멜 케이스 -> 언더스코어 (orderDate -> order_date)

#### TEST 시 
    - @Transaction 
        - 반복 가능한 테스트 지원, 각각의 테스트를 실행할 때마다 트랜잭션을 시작하고 테스트가 끝나면 트랜잭션을 강제로 롤백 (이 어노테이션이 테스트 케이스에서 사용될 때만 롤백)
    - @Runwith(SpringRunner.class) : 스프링과 테스트 통합
    - @SpringBootTest : 스프링 부트 띄우고 테스트(이게 없으면 @Autowired 다 실패)
    

#### `JPA 강점`
    - 데이터 변경 시 entity 안에서, 바뀐 변경 부분(더티 채킹)을 감지해서 변경된 내역들을 데이터베이스에 업데이트 쿼리가 알아서 날아간다.

#### JPA 동적 쿼리
    - String + 연산은 매우 부적절하다.
    - JpaCriteria > 유지보수 매우 안좋음, 코드가 안 읽힘

#### DTO vs Entity 사용
    - 앤티티는 핵심 비즈니스 로직만가지고 있고, 화면을 위한 로직은 없어야한다.
    - 화면이나 API 맞는 DTO를 사용해서 최대한 순수한 엔티티를 유지해야한다.

#### 변경감지(dirty checking)와 병합(merge)
    - 준영속 엔티티
        - 영속성 컨텍스트가 더는 관리하지 않는 엔티티
        - 객체가 이미 DB에 한번 저장되어서 식별자가 존재한다. 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준영속 엔티티로 본다.
    - 준영속 엔티티 수정하는 2가지 방법
        - 병합사용
            - 병합은 모든 필드를 변경해버리고, 데이터가 없으면 `null`로 업데이트 해버린다. 병합을 사용하면 널 문제가 발생하기 때문에 쓰지 않는것이 좋다.
        - 변경감지 기능 사용 
            - 컨트롤러에서 어설프게 엔티티를 생성하지 않고 필요한 데이터만 넘기거나 DTO를 만들어서 넘긴다.
            - 트랜잭션이 있는 서비스 계층에 식별자(id)와 변경할 데이터를 명확하게 전달해야한다.(파라미터 또는 dto)
            - 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경해야한다.
            - 트랜잭션 커밋 시점에 변경이 감지된다.
