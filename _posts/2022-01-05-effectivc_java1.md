---
layout: post
title:  "Effectivce Java - 1"
date:   2022-01-05 11:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 1

# 객체 생성과 파괴

### item1)생성자 대신 정적 팩토리 메서드를 고려하라
- 클래스의 인스턴스를 반환하는 단순한 정적 메서드

```java

public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}

```

- 장점
    - 이름을 가질 수 있다.
        - 생성자 자체만으로는 반환 될 객체의 특성을 설명할 수 없다.
        - 이름만 잘 짓는다면 반환될 객체의 특성을 잘 묘사할 수 있다.
    - 호출할 때마다 인스턴스를 새로 생성하지 않아도 된다.
        - 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체생성을 피할 수 있다.
        - 같은 객체가 자주 요청되는 상황이라면 ?
        - `플라이웨이트 패턴`과 비슷한 기법
        - 인스턴스 통제가 가능하다
            - 인스턴스화 불가
            - 인스턴스가 단 하나임을 보장
    - 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
        - 반환할 객체의 클래스를 자유롭게 선택할 수 있는 `유연성`을 제공한다.
        - API를 만들 때 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.
    - 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환 할 수 있다.
        - 반환타입의 하위 타입이기만 한다면 어떤 클래스의 객체를 반환하든 상관없다.
        - 클라이언트는 해당 존재가 어떻게 되는지 모른다. (캡슐화와 비슷한거 같다.)
    - 정적 팩터리메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
        - (생성자 주입과 비슷한 맥락으로 생각하면 될까?)
- 단점
    - 상속을하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위클래스를 만들 수 없다.
        - 컴포지션 개념이면 장점으로 받아들일 수 있다.
    - 정적 팩터리 메서드는 프로그래머가 찾기 힘들다.

- 흔한 명명 방식
    - from
    - of
    - valueOf
    - instance
    - create
    - getType
    - newType
    - type


### item2) 생성자에 매게변수가 많다면 빌더를 고려하라
- 선택적 매게변수가 많은 때 적절히 대응하기 어렵다.
- 점층적 생성자 패턴도 쓸 수는 있지만, 매게변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽는것이 어렵다.
- 이후 대안 `자바빈즈 패턴` : setter 메서드를 활용
    - 읽기 더 쉬워졌다
    - 단점: 객체 하나를 만들려면 여러번의 메서드를 호출해야한다.
    - 객체가 완전히 생성될 때까지 일관성이 무너진 상태이다.
    - 불변으로 만들 수 없다.
- `빌더패턴`
    - 필요한 객체를 집접 만드는 대신, 필수 매개변수만 생성자를 호출하여 빌더 객체를 얻는다.
    - 빌더는 모두 불변이다.
    - 메서드호출이 흐르듯 연결된다는 뜻으로 플루언트 API, 메서드 연쇄라고 부른다.
    - 명명된 선택적 매개변수를 흉내낸 것
    - 계층적으로 설게된 클래스와 잘 어울리는 패턴이다.
    - 생성자나 정적팩터리가 처리해야할 매개변수가 많을 땐 빌더패턴으로 고려하는게 좋다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Bulider {
        // 필수
        private final int servingSize;
        private final int servings;

        // 선택
        final int calories = 0;
        final int fat = 0;
        final int sodium = 0;
        final int carbohydrate = 0;

        public Bulider(int servingSize, int serving) {
            this.servingSize = servingSize;
            this.serving = serving;
        }

        public Builder calories(int val) {
            this.calories = val;
            return this;
        }

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }

        public Builder sodium(int val) {
            this.sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            this.carbohydrate = val;
            return this;
        }

        private NutirtionFacts(Builder bulider) {
            servingSize = builder.servingSize;
            servings = builder.serving;
            calories = builder.calories;
            fat = builder.fat;
            sodium = builder.sodium;
            carbohydrate = builder.carbohydrate;
        }
    }
}
```

### item3) private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis(){}
    ...

}

```

- public 이나 protected 생성자가 존재하지 않으므로 초기화할 때 만들어진 인스턴스가 전체 시스템에서 하나임을 보장
- 싱글턴임이 API에 명백히 보장
    - public static final
- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글톤을 만드는 가장 좋은 방법이다.


### item4) 인스턴스화를 막으려면 private 생성자를 사용해라
- 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
- private 생성자(); 
- 상속이 불가능하게하는 효과


### item5) 자원을 직적 명시하지말고 의존 객체 주입을 사용하라
- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 클래스가 여러 자원 인스턴스를 지원해야하며, 클라이언트가 원하는 자원을 사용해야한다. 이 조건을 만족하려면 `인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식`

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellCheck(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    ...
}

```

- 불변 보장
- 클라이언트가 의존 객체를 안심하고 사용

정리
> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는것이 좋다.
> 이 자원들을 클래스가 직접 만들게 해서도 안된다. 대신 필요한 자원을 생성자에 넘겨주자. 의존 객체 주입은 클래스의 유연성, 재사용성, 테스트 용이성까지 개선해준다.