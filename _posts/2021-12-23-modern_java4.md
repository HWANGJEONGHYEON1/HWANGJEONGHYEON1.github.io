---
layout: post
title:  "Modern Java - 4"
date:   2021-12-22 14:00:21 +0900
categories: java
---

> 기본에 충실하자 - 4

# Collection API 개선 & 리팩터링, 테스팅, 디버깅

### 맵 팩토리
- Java 9에서 맵을 만들 수 있도록 하는 두가지 방법

```java

Map<String, Integer> map = Map.of("aaa", 1, "bbb", 2, "ccc", 3);

Map<String, Integer> mapEntry = Map.ofEntries(entry("aaa", 1), entry("bbb", 2), entry("ccc",3));
```

### List, Set 메서드 추가
- removeIf: Predicate를 만족하는 요소를 제거(List, Set 구현한 클래스에서 이용가능)

```java
for (Iterator<Transaction> iterator = transactions.iterator(); iterator.hasNext();) {
    Transaction t = iterator().next();
    if (true) {
        iterator.remove(t);
    }
}

// to 

transaction.removeIf(transaction -> true);
```

- replaceAll: 리스트의 각 요소를 새로운 요소로 변경가능.

```java
// 기존 list는 그대로 있고 새로운 리스트를 생성한다.
list.stream()
    .map(String::toUpperCase))
    .collect(Collections.toList());

// to -> 기존 리스트까지 바꾸는 방법
list.replaceAll(data -> data.toUpperCase());

```

### 맵 처리
- forEach
    - 맵에서 key, value를 반복하며 확인을 하려면 Map.Entry<Key, Value>의 반복자를 활용해야한다.
    - Java 8 부터는 Biconsumer를 인수로 받는 forEach 메서드가 추가되어 간편하게 구현할 수 있다.

```java
Map<String, Integer> map = new HashMap<>();
map.forEach((k, v) -> log.info("k : {}, v = {}", k, v);

```

- 정렬 메서드
    - Entry.comparingByValue
    - Entry.comparingByKey

```java
    Map<String, Integer> map = Map.ofEntires(entry("aaa", 1), entry("bbb", 2), entry("ccc",3));
    map.entrySet()
        .stream()
        .sorted(Entry.comparingByKey());
```
- getOrDefault
    - 맵에 존재하지 않는 키로 값을 찾으면 NPE 발생하기 때문에 Null 체크필요

```java
Map<String, Integer> map = Map.ofEntires(entry("aaa", 1), entry("bbb", 2), entry("ccc",3));
map.getOrDefault("aaa", "ddd") // "aaa"
map.getOrDefault("qwe", "ddd") // "ddd"
```

### 가독성과 유연성을 개선하는 리팩토링
- 람다, 메서드참조, 스트림 등의 기능을 이용해 가독성을 높이고 유연한 코드로 리팩토링
- 익명 클래스를 람다표현식으로 리팩토링
    - 모든 익명클래스를 람다 표현식으로 바꿀 수 있는건 아님
    - 익명 클래스에서 사용한 this, super는 서로 다른 의미
    - this는 익명클래스 자신을 가르키지만 람다에서 this는 람다를 감싸는 클래스를 가르킨다.

```java
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello");
            }
        };
        // to
        Runnable r2 = () -> System.out.println("Hello");

```

- 람다 표현식을 메서드 참조로 리팩토링
    - 람다 표현식을 메서드 참조로 바꾸면 가독성을 높이고 코드의 의미를 명확하게 표현할 수 있다.
    - 예시로 밑의 클래스에서 dish -> 블록을 Dish 클래스에 메서드로 변경하자(1)
    - 2로 깔끔하게 변경할 수 있다.


```java

Map<CaloricLevel, List<Dish>> dishByCaloricLevel = menu.stream()
                .collect(groupingBy(dish -> {
                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                    else return CaloricLevel.FAT; //  1.
                }));
// to
Map<CaloricLevel, List<Dish>> dishByCaloricLevel = menu.stream()
                .collect(groupBy(Dish::getCaloricLevel));// 2.
```

- comparingBy와 maxBy같은 정적 헬퍼 메서드 활용

```java
    animal.sort(Tiger tiger1, Tiger tiger2) -> tiger1.getWeight().compareTo(tiger2.getWeight());
    //to
    animal.sort(comparing(Tiger::getWeight));
```

### 람다로 객체지향 디자인패턴 리팩터링
- 람다를 이용하여 이전에 디자인 패턴으로 해결하던 문제를 더 간단하게 해결할 수 있다.
- 람다 표현식으로 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재 구현할 수 있다.
- 람다 표현식으로 기존의 많은 객체지향 디자인 패턴을 제거하거나 간결하게 재구현 할 수 있다.
- 디자인패턴
    - 전략
    - 템플릿 메소드
    - 옵저버
    - 의무체인
    - 팩토리


### 디자인패턴
- 전략패턴
    - 한 유형의 알고리즘을 보유한 상태에서 런타임 시 적절한 알고리즘을 선택하는 기법
    - 클라이언트 -> 전략 -> A,B(런타임 상황)으로 구성
    - Strategy 인터페이스, StrategyA, StrategyB, 클라이언트

```java

class IsString implements  ValidationStrategy {

    @Override
    public boolean execute(String s) {
        return s.matches("[a-z]+");
    }
}

class isNumeric implements  ValidationStrategy {

    @Override
    public boolean execute(String s) {
        return s.matches("[0-9]+");
    }
}

class Validator {
    private final ValidationStrategy validationStrategy;
    
    public Validator(ValidationStrategy validationStrategy) {
        this.validationStrategy = validationStrategy;
    }
    
    public boolean validate(String s) {
        return validationStrategy.execute(s);
    }
}

class Client {
    Validator numericValidator = new Validator(new isNumeric());
    numericValidator.validate("aaa");

    Validator stringValidator = new Validator(new IsString());
    stringValidator.validate("bbbb");
}

```

- 인터페이스만 선언하고 구현체를 작성할 필요없이

```java
public class Client {
    public static void main(String[] args) {
        Validator numericValidator = new Validator(s -> s.matches("[a-z]+"));
        boolean validate = numericValidator.validate("bbb");

        Validator stringValidator = new Validator(s -> s.matches("[0-9]+"));
        boolean bbbb = stringValidator.validate("123");
    }
}
```

- 템플릿 메서드
    - 특정 기능의 일부만 변경하여 사용할 수 있는 유연함을 제공하는 패턴

```java

public abstract class Glass {
    
    public void see(GlassType type, Consumer<OptionGlass> glass) {
        System.out.println("이 안경은 " + option() + " 입니다");
    }

    protected abstract String option(); 
}

new BlueLightGlass().option(BLUELIGHT, (OptionGlass glass) -> glass.see());

```

- 팩토리
    - 인스턴스화 로직을 클라이언트에 노출하지 않고 객체를 만들기 위해 팩토리 디자인 패턴을 사용한다.

```java

    public static Product createProduct(String name) {
        if (name.equals("loan")) {
          return new Loan();
        }

        if (name.equals("stock")) {
            return new Stock();
        }

        if (name.equals("Bond")) {
            return new Bond();
        }

        throw new IllegalArgumentException("상품을 생산할 수 없습니다.");
    }

    
    final static Map<String, Supplier<Product>> map = new HashMap<>();
    static {
        map.put("loan", Loan::new);
        map.put("stock", Stock::new);
        map.put("bond", Bond::new);
    }

    public static Product createProduct(String name) {
        Supplier<Product> productSupplier = map.get(name);

        if (Objects.nonNull(productSupplier)) {
            return productSupplier.get();
        }
        throw new IllegalArgumentException("상품을 생산할 수 없습니다.");

    }
```