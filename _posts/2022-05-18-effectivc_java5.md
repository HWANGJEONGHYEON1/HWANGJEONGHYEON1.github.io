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
- 필요한 용도에 맞는게 있따면, 직접 구현하지말고 표준 함수형 인터페이스를 활용


|---|---|---|
|인터페이스|함수 시그니처| 예|
|UnaryOperator<T> | T apply(T t) | String::toLowerCase|
|BinaryOperator<T> | T apply(T t1, T t2) | BigInteger::add | 
|Predicate<T> | boolean test(T t)| Collection::isEmpty()|
|Function<T, R>| R apply(T t)| Arrays::asList|
|Supplier<T>| T get() | Instant::now|
|Consumer<T> | void accept(T t)| System.out::println|

- Comparator 
    - 자주 쓰이며 이르 자체가 용도를 명확히 설명해준다.
    - 반드시 따라야하는 규약이 있다.
    - 유용한 디폴트 메서드를 제공할 수 있다.
- 의도를 명시하는 세가지 목적
    - 해당 클래스의 코드나 설명 문서를 읽을 이에게 인터페이스가 람다용으로 설계된 것임을 알려준다.
    - 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
    - 결과를 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
    - 직접 만든 함수형 인터페이스에서는 @FunctionalInterface 애노테이션을 사용하라.


## itemt45) 스트림은 주의해서 사용하라
- 스트림과 반복 중 어느쪽이 나은지 확신하기 어렵다면 둘 다 작성해보고 나은쪽을 선택하라.

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
    
    - 스트림, 람다, 메서드 참조를 사용했지만 스트림 코드를 가장한 반복적 코드다.
    - 같은 기능의 반복적 코드보다 길고, 읽기 어렵고, 유지보수에도 좋지않다.
    - forEach 연산은 스트림 계산 결과를 보고할 때만 사용하고 계산할 때는 쓰지말자.
    - 수집기
        - toList()
        - toSet()
        - toCollection(collectionFactory)
    
    ```java

            Map<String, CoinColor> collect = Stream.of(CoinColor.values())
                .collect(
                        toMap(Object::toString, e -> e));

            // 인수 3개를 받는 toMap은 어떤키와 그 키에 연관된 원소들 중 하나를 골라 연관 짓는 맵을 만들 때 유용하다.
            // maxBy는 Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다.
            // 이 경우 비교자 생성 메서드인 comparing이 maxBy에 넘겨줄 비교자를 반환하는대, 자신의 키 추출함수로는 Albums::sales를 받았다.
            // 복잡해 보일 수 있지만 잘 읽힌다.
            albums.collect(
                toMap(Album::artist, a->a, maxBy(Album::sales));

            
            // 각 사이즈가 키가 되고 사이즈가 비슷한 것 들끼리 리스트가 만들어진다.
            List<String> words = Arrays.asList("abc", "defc", "aa", "aaa", "bbbb", "cc", "dddddd");
            words.stream()
                .collect(groupingBy(word -> alphabetsize(word)))
                .forEach((k, v) -> System.out.println(k + " " + v));
            
            ... 
            private static int alphabetsize(String word) {
                return word.length();
            }
    }
    ```

## item47) 반환타입으로는 스트림보다 컬렉션이 낫다.
- 원소 시퀀스를 반환하는 공개 API 반환타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.
- Arrays.asList, Stream.of 메서드로 쉽게 반복과 스트림을 지원할 수 있다.
- 하지만 반환하는 시퀀스의 크기가 메모리에 올려도 안전하다면 그대로 올려도 좋지만, 컬렉션을 반환한다는 이유로 큰 시퀀스를 메모리에 올리는 것은 안된다.
- 원소시퀀스를 반환하는 메서드를 작성할 때는, 스트림으로 처리하기를 원하는 사용자와 반복으로 처리하길 원하는 사용자가 모두 있을 수 있음을 떠올리고, 양쪽을 다 만족시키려 노력해야한다.
- 이미 원소들을 컬렉션에 담아 관리하고 있거나 컬렉션을 하나 더 만들더라도 될 정도로 원소개수가 적다면 ArrayList 같은 표준 컬렉션에 반환하라.
