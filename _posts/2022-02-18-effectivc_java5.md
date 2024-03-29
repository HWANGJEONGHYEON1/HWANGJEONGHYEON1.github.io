---
layout: post
title:  "Effectivce Java - 5"
date:   2022-02-18 09:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 4

# 제네릭

### item32) 제네릭과 가변변수를 함께 쓸 때는 신중하라
- 가변인수와 제네릭은 궁합이 좋지않다.
- 가변인수 기능은 배열을 노출하여 추상화가 완벽하지 못하여 배열과 제네릭의 타입 규칙이 서로 다르다.
- 제네릭 varargs 매게변수는 타입 안전하지는 않지만, 허용된다.
- 메서드에 제네릭 varargs 매게변수를 사용하고자 한다면, 그 메서드가 타입 안전한지 확인한 다음 @SafeVarargs를 사용하자

### item33) 타입 안전 이종 컨테이너를 고려하라
- 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어있다.
- 하지만 컨테이너 자체가 아닌 키를 매게변수로 바꾸면 이런 제약이 없는 타입 이종 컨테이너를 만들 수 있다.
- 타입 안정 이종 컨테이너는 Class를 키로 쓰며, 이런식으로 쓰이는 Class 객체를 타입 토큰이라고 한다.


# 열거타입과 애노테이션

### item34) int 상수 대신 열거 타입을 사용하라
- 정수 열거 타입은 타입 안전을 보장할 방법이 없으며, 표현력도 좋지 않다.
- 자바의 열거 타입은 위의 패턴의 단점을 보완하고, 여러 장점을 안겨준다.
- 열거타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 만들어 `public static final` 필드로 공개한다.
- 열거 타입은 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 `final`이다.
- 인스턴스들은 딱 하나씩 존재가 보장된다.
    - 열거타입은 인스턴스 통제된다.
    - 싱글턴은 원소가 하나 뿐인 열거타입이라 할 수 있다.
    - 컴파일 타임의 타입의 안전성을 보장한다.
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.

### item35) ordinal 메서드 대신 인스턴스 필드를 사용하라
- 열거타입 상수에 연결된 값은 ordinal 메서드로 얻지말고 인스턴스 필드를 저장하자.

```java
public enum Ensemble {
    A(1),B(2)...;

    private final int numberOfMusicians;
    Ensemble(int size) {
        this.numberOfMusicians = size;
    }

    public int numberOfMusicians(){return numberOfMusicians}
}

```

### item36) 비트 필드 대신 Enumset을 사용하라


### item37) ordinal indexing 대신 EnumMap을 사용하라 
- 배열의 인덱스를 얻기위해 ordinal을 쓰는것은 일반적으로 좋지 않으니, EnumMap을 사용하라.


### item37) 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라.
- 열거타입은 타입은 타입 안전하지만, 확장하면 할 수록 설계와 구현이 복잡해진다.
- 열거 타입이 인터페이스를 구현할 수 있다.
- 연산코드용 인터페이스를 정의하고 열거 타입이 인터페이스를 구현하게 하면 된다.

```java
public enum BasicOperation implements Operation {
    PLUS("+") {
        @Override
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        @Override
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        @Override
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        @Override
        public double apply(double x, double y) {
            return x / y;
        }
    };
    
    private final String symbol;
    
    BasicOperation(String symbol) {
        this.symbol = symbol;
    }
}
```

- 열거타입인 BasicOperation 확장 불가능하지만, Operation은 확장할 수 있다.

```java


public enum ExtendedOperation implements Operation {
    
    EXP("^") {
        @Override
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMINDER("%") {
        @Override
        public double apply(double x, double y) {
            return x % y;
        }
    };
    
    private String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }
}


```

- Operation을 구현한 열거타입만 작성하면 끝이다.
- 열거타입은 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거타입을 함께 사용해 같은 효과를 낼 수 있다.


### item39) 명명 패턴보다 애너테이션을 사용하라
