---
layout: post
title:  "Injection 종류"
date:   2021-04-05 19:22:21 +0900
categories: spring
---

> ```Dependency injection```

- 런타임에 의존관계를 결정할 수 있다.

### 생성자 주입
  - 생성자 호출시점에 딱 1번만 호출되는것이 보장 ```불변, 필수```
  - 외부에서 객체를 변경할 수 있는 방법이 없다.
  - 생성자가 한 개 있으면 autowired 생략가능.
  - 빈을 등록하면서 자동 주입이 일어남.

### setter 주입
 - ```선택, 변경``` 가능성이 있는 의존관계에 사용
 - @Autowired의 기본동작은 주입할 대상이 없으면 오류발생

### 필드 주입
 - 필드에 바로 주입하는 방식
 - 코드가 간결
 - 외부에서 변경이 불가능하여 테스트하기 힘듬(치명적)
   - 원하는 mock 데이터로 주입하여 테스트를 하고싶은대 할 방법이 없음.(결국 setter 사용)
 - DI프레임워크가 없으면 불가능.

### 일반 메서드 주입
 - 일반 메서드를 통해 주입받을 수 있음.
 - 한번에 여러 필드를 주입받을 수 있음.

## 생성자 주입 선택하는 이유

- <b>final</b> 키워드를 넣을 수 있다.
- Immutable
  - 의존관계주입이 일어나면 어플리케이션 종료시점까지 의존관계를 변경할 일이 없어야한다(불변)
  - 실수로 변경할 수도있고, 변경하면 안되는 메서드를 열어두는 것은 객체지향에 좋지 않다.
  - 생성자 주입은 생성할 때 1번만 호출되므로 이후에 호출되는 일이 없다. (불변)   
  - 생성자에서 실수로 값이 설정되지 않는 오류를 컴파일 시점에서 막아준다.
  - 생성자 주입을 제외한 주입들은 모두 생성자 이후에 호출되므로, 필드에 <b>final</b> 사용할 수 없다.
- 프레임워크에 의존하지 않고 순수한 자바 언어의 특징을 잘 살릴 수 있는 방법


## 스프링 빈 라이프사이클
- Load Bean Definition : 스프링 컨테이너를 시작할 때 먼저 설정파일을 통해 DI를 할 Bean 정보를 BeanDifinition으로 생성한다.
- DI 단계: 설정 파일을 통해 읽은 BeanDifinition 정보를 바탕으로 Bean 인스턴스를 생성하고 의존관계를 연결한다.