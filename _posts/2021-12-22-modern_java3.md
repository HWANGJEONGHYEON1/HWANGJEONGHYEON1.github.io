---
layout: post
title:  "Modern Java - 3"
date:   2021-12-22 14:00:21 +0900
categories: java
---

> 기본에 충실하자 - 3

# Stream 활용

### Predicate를 활용한 스트림
- filter(Predicate)
- 스트림 슬라이싱

```java
Stream<T> filter(Predicate<? super T> predicate);

Stream<T> distinct();

/**
* Predicate 결과가 True인 요소에 대한 필터링 Predicate이 처음으로 거짓이 되는 지점에 연산을 멈춤
*/
Stream<T> takeWhile(Predicate<? super T> predicate);

/**
* Predicate 결과가 False 요소에 대한 필터링 Predicate이 처음으로 거짓이 되는 지점까지 발견된 요소를 버린다.
*/
Stream<T> dropWhile(Predicate<? super T> predicate);
```

### 스트림 축소, 스킵

```java

/**
* 주어진 값 이하의 크기를 갖는 새로운 스트림을 반환
*/
Stream<T> limit(long maxSize); 

/**
* 처음부터 n개 까지 스킵하고 나머지 요소를 반환
*/
Stream<T> skip(long n);
```

### MAP
- map: 함수를 인수로 받아 새로운 요소로 매핑된 스트림을 반환. 기본형 요소에 대한 mapTo${Type} 메서드 지원(mapToInt, mapToLong ..)
- flatMap: 제공된 함수를 각 요소에 적용하여 새로운 하나의 스트림으로 매핑. 결과적으로 하나의 평면화된 스트림 반환

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extedns R>> mapper);
```

### 검색과 매칭
- anyMatch: 적어도 한 요소와 일치하는지 확인하는지(일치하면 바로 true 리턴)
- allMatch: 모든 요소가 일치하는지 검사(일치하지 않으면 바로 false 리턴)
- noneMatch: 모든 요소가 일치하지 않는지 검사(일치하는 순간 false 리턴)
- findFrist: 첫번째 요소를 찾아 반환(순서가 정해져있을 떄)
- findAny: 요소를 찾으면 반환(순서가 상관없을 떄)

```java
boolean anyMatch(Predicate<? super T> predicate);
boolean allMatch(Predicate<? super T> predicate);
boolean noneMatch(Predicate<? super T> predicate);
Optional<T> findFrist();
Optional<T> findAny();
```

### 리듀싱
- reduce: 모든 스트림 요소를 BinaryOperator로 처리해서 값으로 도출 => 두번 째 reduce 메서드와 같은 경우 초기값이 없으므로 아무요소가 없을 때 `Optional<T>` 반환
- reduce 메서드 장점: 기존의 단계적 반복으로 구하는것보다 reduce를 이용하면 내부 반복이 추상화되면서 내부 구현에서 병렬로 reduce를 실행할 수 있게 된다. 반복적인 합계에서는 sum 변수를 공유해야하므로 병렬하기 어렵다.
- max, min: 요소에서 최대값과 최소값을 반환. 빈 스트림일 수 있기에 Optional<T>
```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);

Optional<T> max(Comparator<? super T> comparator);
Optional<T> min(Comparator<? super T> comparator);
```

### 숫자형 스트림
- 숫자 스트림을 효율적으로 처리할 수 있도록 기본형 특화 스트림을 제공
- IntStream, DoubleStream, LongStream이 존재하며 각각의 인터페이스에는 숫자 스트림의 합계를 계산하는 Sum, 최대값 요소를 검색하는 max 같이 자주 사용하는 숫자 관련 리듀싱 연산 메서드를 제공

# 스트림 데이터 수집(Collector)

- 스트림 요소를 어떤식으로 도출할지 지정
- 훌륭하게 설계된 함수형 API 장점으로는 높은 수준의 조합성과 재사용성을 꼽을 수 있다.
- Collectors에서 제공하는 메서드의 기능은 3가지로 구분
    - 스트림 요소를 하나의 값으로 리듀싱하고 요약
    - 요소 그룹화
    - 요소 분할
- 리듀싱과 요약
    - counting
    - maxBy, minBy: 최대 혹은 최소를 만족하는 요소를 찾는다.
    - summingInt: 객체를 int로 매핑하는 인수를 받아 합을 계산한다.
    - averaginInt
    - joining: 내부적으로 StringBuilder를 이용하여 문자열을 하나로 만든다.
- 그룹화
    - 그룹화 함수는 어떤 기준으로 스트림을 분류하는 속성을 가졌기에 분류함수라고 부른다.
    - groupingBy

```java
// 분할 함수
public static <T, K> Collector<T, ?, Map<K, List<T>>> groupingBy(
    Function<? super T, ? extends K> classifier) {

    return groupingBy(classifier, toList());
}

// 분할 함수, 감싸인 컬렉터
public static <T, K, A, D> Collector<T, ?, Map<K, D>> groupingBy(
    Function<? super T, ? extends K> classifier,
    Collector<? super T, A, D> downstream) {

    return groupingBy(classifier, HashMap::new, downstream);
}

// 분할 함수, 반환 타입, 감싸인 컬렉터
public static <T, K, D, A, M extends Map<K, D>> Collector<T, ?, M> groupingBy(
    Function<? super T, ? extends K> classifier,
    Supplier<M> mapFactory,
    Collector<? super T, A, D> downstream) {
  // ...
}

/**
*  컬렉터를 중첩할 시 가장 외부계층 안쪽으로 다음과 같은 작업이 수행된다.
*  1. groupingBy에서 분류하는 요소로 Dish::type에 따라 서브 스트림을 그룹화
*  2. groupingBy 컬렉터는 collectingAndThen으로 감싼다. 그래서 두 번째 컬렉터는 그룹화된 서브스트림에 적용된다.
*  3. 리듀싱 컬렉터(maxBy)가 서브스트림에 연산을 수행한 결과에 Optional::get 변환 함수가 적용된다.
*  4. groupingBy 컬렉터가 반환하는 맵의 분류가 키이고, 대응하는 value는 Dish의 가장 높은 칼로리이다.
*/

Map<Dish.Type, Dish> mostCaloricByType = 
    menu.stream()
        .collect(groupingBy(Dish::Type, // 분류 함수
                collectingAndThen(
                    maxBy(comparingInt(Dish::getCalories)), // 감싸인 컬렉터
                Optional::get)); // 변환 함수
```

### 컬렉터 인터페이스

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```

- T는 수집될 항목의 제네릭 형식
- A는 누적자, 수집 과정에서 중간 결과를 누적하는 객체의 형식
- R은 수집 연산 결과 객체의 형식
- Supplier 메서드: 새로운 결과 컨테이너 만들기
    - supplier는 수집 과정에서 빈 누적자 인스턴스를 만드는 파라미터가 없는 함수
- accumulator: 결과 컨테이너에 요소 추가
    - 누적자(스트림으 n - 1개 항목을 수집한 상태)와 n번째 요소를 함수에 적용한다,
- finisher: 스트림 탐색을 끝내고 누적자 객체를 최종 반환하면서 누적 과정을 끝낼 떄 호출할 함수를 반환해야 한다.
- combiner: 스트림의 서로 다른 서브파트를 병렬로 처리할 때 누적자가 이 결과를 어떻게 처리할 지 정의(BinaryOperator)