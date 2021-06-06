---
layout: post
title:  "정적 팩토리 메서드"
date:   2021-06-05 22:40;
categories: java, design-pattern
tags: [java, design-pattern]
---

## static 메서드로 객체 생성을 캡슐화

### 장 단점
    - 장점
        1. 가독성
        2. 새로운 객체를 생성할 필요 없다.
        3. 하위 자료형 객체 반환 가능
    - 단점
        1. 정적 팩토리 메서드만 있는 클래스는 생성자가 없으므로 하위 클래스를 만들지 못한다.

### 가독성
- `이름을 가질 수 있다.`

```java
    public OrderGoods(Goods goods, int orderPrice, int count) {
        this.goods = goods;
        this.orderPrice = orderPrice;
        this.count = count;
    }
    
```

```java
    public static OrderGoods createOrder(Goods goods, int orderPrice, int count) {
        return OrderGoods.builder()
                .goods(goods)
                .orderPrice(orderPrice)
                .count(count)
                .build();
    }
```

 ### Ref.
* <https://velog.io/@ljinsk3/%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C>
* <https://johngrib.github.io/wiki/static-factory-method-pattern/>