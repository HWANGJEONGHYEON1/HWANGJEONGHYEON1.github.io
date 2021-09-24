---
layout: post
title:  "Spring bean이 필요한 경우"
date:   2021-04-09 19:22:21 +0900
categories: spring
---

#  Bean이 모두 필요한 경우

## 빈을 List, Map에 담기
```java
 @Test
    void findAllBean() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class,DiscountService.class);


        DiscountService discountService = ac.getBean(DiscountService.class);
        final Member userA = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(userA, 10000, "fixDiscountPolicy");
        assertThat(discountService).isInstanceOf(DiscountService.class);
        assertThat(discountPrice).isEqualTo(1000);
    }

    static class DiscountService {
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policyList;

        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policyList) {
            this.policyMap = policyMap;
            this.policyList = policyList;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policyList = " + policyList);
        }

        public int discount(Member userA, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(userA, price );
        }
    }
```

- DiscountService는 Map으로 모든 DiscountPolicy를 주입받는다.
- DiscountPolicy는 현재 인터페이스
- ixDiscountPolicy , rateDiscountPolicy가 구현 클래스이며, bean에 두 개가 주입된다.
- map의 키에 bean 이름을 너헝주고, 그 값을 DiscountPolicy 타입으로 조회한 모든 스프링 bean을 담아준다.

### 조회한 빈이 모두 필요할 때, List,Map을 사용하여 전략패턴을 사용하여 할 수 있는 장점이 있다.

### Component vs bean
- bean : 외부 라이브러리를 등록하고 싶을 때 빈을 사용(클래스 레벨 사용 불가)
- component : 개발자가 직접 컨트롤하려고 할 때 사용(클래스 레벨 사용)
- 
## 정리
- Bean : @Bean은 수동으로 스프링 컨테이너에 등록할 스프링 빈을 직접 등록
- Component: @Component는 자동으로 스프링 컨테이너에 스프링 빈을 등록
- 자동으로 빈 등록해주는 것을 기본으로 사용
- 직접 등록하는 기술지원 객체는 수동으로 등록(외부 라이브러리)
- 다형성을 활용하는 비즈니스 로직은 수동 등록을 고민
