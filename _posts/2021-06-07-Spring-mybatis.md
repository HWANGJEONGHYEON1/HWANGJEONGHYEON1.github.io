---
layout: post
title:  "MyBatis 기본생성자"
date:   2021-06-07 19:50
categories: java, spring, mybatis
tags: [java, spring, mybatis]
---

## MyBatis 기본생성자가 필수이다.

> 빌더 패턴을 사용하여 테스트 코드를 작성 중 에러 직면

### 개인 프로젝트 진행 중, 정적 메서드 팩토리를 사용
    - 소스
    - 정적 메서드 팩토리에서 빌더 패턴을 사용할 경우 기본 생성자를 허용하지 않는다.
```java

@Builder
@Getter
public class OrderVO {

    private Long id;

    private Long userId;

    private Long goodsId;

    private OrderStatus state;

    private String address;

    private List<OrderGoodsVO> orderGoodsList;

    private Date orderDate;

    public static OrderVO createOrder(Long userId, String addr, List<OrderGoodsVO> orderGoods) {
        return OrderVO.builder()
                .userId(userId)
                .state(OrderStatus.ORDER)
                .address(addr)
                .orderGoodsList(orderGoods)
                .build();
    }

    public void addGoods(OrderGoodsVO orderGoods) {
        orderGoodsList.add(orderGoods);
    }

    public void cancel() {
        this.state = OrderStatus.CANCEL;
    }

    public int getTotalPrice() {
        int totalPrice = 0;
        totalPrice = orderGoodsList.stream()
                .mapToInt(OrderGoodsVO::getTotalPrice)
                .sum();

        return totalPrice;
    }
}
```

> 에러내용
~~~
Error attempting to get column 'address' from result set.  
Cause: java.sql.SQLDataException: Cannot determine value type from string 'suwon'
~~~

### 이유
- MyBatis는 기본적으로 기본생성자가 `필수`이다.
- MyBatis의 ResultMapping은 ObjectFactory를 사용하여 객체의 초기화가 필요하고, 기본 생성자를 먼저 검색하여 사용

### 해결
    - 해결은 했지만 잘한건지는 모르겠다.
    - @Builder, @Getter, @AllArgsConstructor, @NoArgsConstructor 다 추가



 ### Ref.
* <https://velog.io/@ljinsk3/%EC%A0%95%EC%A0%81-%ED%8C%A9%ED%86%A0%EB%A6%AC-%EB%A9%94%EC%84%9C%EB%93%9C%EB%8A%94-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C>
* <https://johngrib.github.io/wiki/static-factory-method-pattern/>