---
layout: post
title:  "Effectivce Java - 5"
date:   2022-02-04 16:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 4

# 열거타입과 애노테이션

### item39) 명명패턴보다 어노테이션으로 사용하라
- 명명패턴 단점
    1. 오타가나면 안된다.
        - junit3에서는 실수로 이름을 잘못지으면 메서드를 그냥 지나간다.(테스트 실패 성공여부 모름)
    2. 올바른 프로그램 요소에만 사용되리라 보증할 방법이 없다.
    3. 프로그램 요소르르 매개변수로 전달할 마땅한 방법이 없다.
- 애노태이션
    - 위의 모든 문제를 모두 해결해주며, Junit4부터 전면 도입되었다.
    - 
```java
/**
테스트 메서드임을 선언하는 메서드
매개변수 없는 정적 메서드 전용
*/

// @Test가 런타임에도 유지되어야한다는 뜻
@Retention(RetentionPolicy.RUNTIME)
// @Test가 반드시 메서드 선언에만 사용돼어야함. 즉, 클래스 변수에는 선언할 수 없다.
@Target(Element.METHOD)
public @interface Test {

}
```

```java
// 명시한 예외를 던져야만 성공하는 테스트 메서드용 애노테이션
@Retention(RetentionPolicy.RUNTIME)
// @Test가 반드시 메서드 선언에만 사용돼어야함. 즉, 클래스 변수에는 선언할 수 없다.
@Target(Element.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value(); // Throwable을 확장한 클래스 객레이며, 모든 예외 타입을 다 수용
}

```


## item40)  @Override 어노테이션을 일관되게 사용하라
- 상위 클래스의 메서드를 재정의하려는 모든 매서드에서 @Override 애노테이션을 달자
- 어노테이션을 달고 컴파일을 하면 컴파일오류로 미리 알 수 있다.
- 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우엔 달지 않아도 된다.

## item41) 정의하려는 타입이라면 마커인터페이스를 사용하라
- 마커 인터페이스는 이를 구현한 클래스의 인스턴스를 구분하는 타입으로 쓸 수 있으나, 마커 애노테이션은 그렇지 않다.
- 마커 인터페이스가 나은점은 적용 대상을 더 정밀하게 지정할 수 있다.


# 람다와 스트림
> 자바 8에서 함수형 인터페이스, 람다, 메서드 참조라는 개념이 추가


## item42) 익명 클래스보다는 람다를 사용하라
- 어떤 동작을 하는지 명확하게 드러난다.
- 반환 값의 타입이 언ㄱ브되지 않았지만, 컴파일러가 문맥을 보고 타입을 추론해준것
- 타입을 명시해야하 코드가 더 명확할 때만 제외하고는, 람다의 매게변수 타입은 생략하자
- `메서드나 클래스와 달리, 람다는 이름이 없고 문서화도 못한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지말아야한다.`


```java

    List<String> words = Arrays.asList("aa", "bbb", "cccccc");
    Collections.sort(words, new Comparator <String>() {

        @Override
        public int compare(String o1, String o2) {
            return Integer.compare(o1.length(), o2.length());
        }
    });
    
    Collections.sort(words, (o1, o2) -> Integer.compare(o1.length(), o2.length()));

    Collections.sort(words, Comparator.comparingInt(String::length));

    public static enum Operation {
        PLUS("+", (x, y) -> x + y),
        MINUS("-" , (x, y) -> x - y);

        private final String symbol;
        private final DoubleBinaryOperator op;

        Operation(String symbol, DoubleBinaryOperator op) {
            this.symbol = symbol;
            this.op = op;
        }

        public double apply(double x, double y) {
            return op.applyAsDouble(x, y);
        }
    }

```

## item43) 람다보다는 메서드 참조를 사용하라
- 람다가 익명 클래스보다 나은 점 중에서 가장 큰 특징은 간결함
- 함수 객체를 람다보다도 더 간결하게 만드는 방법이 있으니, 메서드 참조이다.
- map.merge(key, 1, (count, incr) -> count + incr);
    - 머지 메서드는 키 값 함수를 인수로 받으며, 주어진 키가 맵 안에없다면 주어진 쌍을 그래도 저장.
    - 키가 있다면 함수를 주어진 현재 값에 적용한다음 그 결과로 현재 값을 덮어쓴다.
    - map.merge(key, 1, Integer::sum;
    
    |메서드 참조 유형|예 | 같은 기능을 하는 람다|
    |---|---|---|---|
    | 정적 | Integer.::parseInt | str -> Integer.parseInt(str)|
    | 한정적(인스턴스) | Instant.now()::isAfter()| Instant then = Instant.now(); t -> then.isAfter(t)|
    | 비한정적(인스턴스) | String::toLowerCase | str -> str.toLowerCase() |
    | 클래스 생성자 | TreeMap<K,V>::new | () -> new TreeMap<K,V>() |

- 메서드 참조는 람다의 간단 명료한 대안이 될 수 있다. 메서드 참조 쪽이 짧고 명확하다면 메서드 참조쓰고, 그렇지 않을때만 람다를 사용하라

## item44) 표준 함수형 인터페이스를 사용하라
- 

## item46) 스트림에서는 부작용없는 함수를 사용하라
- 순수함수란?
    - 오직 입력만이 결과에 영향을 주는 함수를 말한다.
    - 다른 가변상태를 참조하지 않고, 함수 스스로도 다른 상태를 변경하지 않는다.

    ```java
    Map<String, Long> freq = new HashMap<>();
    try (Stream<String> words = new Scanner(file).tokens()) {
        words.forEach(word -> {
            freq.merge(word.toLowerCase(), 1L, Long::sum);
        });
    }
    ```
    
    - 스트림, 람다, 메서드 참조를 사용했지만 