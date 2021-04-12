---
layout: post
title:  "autoWired"
date:   2021-04-08 19:22:21 +0900
categories: spring
---

## autowired
 - 타입매칭을 시도하고, 타입 매칭 결과가 2개 이상일 때 필드명으로 빈 이름 매칭


## Qualifier
  - @Qualifier는 @Qualifier 찾는 용도로만 쓰는게 좋음
  - Qualifier 끼리 매칭
  - 빈이름 매칭 
  - 안되면 NoSuchBeanDefinitionException 발생
  - 예제
  ```java
    @Component
    @Qualifier("mainDiscountPolicy")
    public class DiscountPolicy ...
  ```
  ```java
  public OrderServiceImpl (@Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy)
  ```

## Primary
 - 우선순위를 정하는 방법, @Autowired 시 여러번 매칭되면 @Primary가 우선권을 가진다