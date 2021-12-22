---
layout: post
title:  "Modern Java - 1"
date:   2021-12-21 13:00:21 +0900
categories: java
---

> 기본에 충실하자 - 1

### 자바 함수
- 프로그래밍 언어의 핵심은 `값을 바꾸는 것` => 일급 값값
- 런타임에 메서드를 전달할 수 있다면 프로그래밍에 유용하게 활용될 수 있다.

### Method Reference
- 동작의 전달을 위해 익명 클래스를 만들고 메서드를 구현해서 넘길필요 없이, `::`를 이용해서 전달할 수 있다.
```java
File[] hiddenFiles = new File(".").listFiles(new FileFilter() {
    public boolean accept(File file) {
        return file.isHidden();
    }
});
// 위의 코드가 밑의 코드로

File[] hiddenFiles = new File(".").listFiles(File::isHidden);

```

- 명시적으로 메서드 명을 참조함으로써 가독성을 높일 수 있다.
- 메서드 참조의 3가지 유형
    - 정적 메서드 참조: Integer::parseInt
    - 다양한 형식의 인스턴스 메서드 참조: String::length
    - 기존 객체의 인스턴스 메서드 참조: Animal::getWeight
    - 딴 예로 ClassName::new 처럼 new 키워드를 사용하여 기본생성자를 참조할 수 있다.
- 람다 표현식을 조합하는 유용한 메서드
    - Comparator
        - comparing: 비교에 사용할 Function 기반의 키 지정
        - reversed: 역정렬
        - thenComparing: 동일한 조건에 대한 추가적인 비교
    - Predicate
        - and: and 연산
        - or: or
        - negate: not
    - Function
        - andThen: 이후에 처리할 function 추가
        - compose: 이전에 처리되어야할 function 추가

### lamDa

- 자바 8에서는 일급값으로 취급할 뿐 아니라 람다를 포함하여 함수도 값으로 취급할 수 있다.
- 특징
    - 익명: 보통의 메서드와 달리 이름이 없으므로 익명이라고 한다. `구현해야할 코드에 대한 걱정거리가 줄어든다.`
    - 함수: 람다는 메서드처럼 특정 클래스에 종속되지 않으므로 함수라 부른다.(메서드처럼 파라미터 리스트,바디, 반환 값, 예외리스트는 포함)
    - 전달: 람다 표현식을 메서드 인수로 전달하거나 변수로 저장할 수 있다.
    - 간결성: 익명클래스처럼 많은 자질구레한 코드를 구현할 필요가 없다.
- 람다의 구성

```
/**
*   -       람다의 파라미터         -          =             람다바디                =
* (Animal animalA, Animal animalB) -> animalA.getWeight().compareTo(animalB.getWeight());
*/

```
- 함수형 인터페이스
    - 정확히 하나의 추상 메서드를 지정하는 인터페이스.
    - 람다 표현식으로 함수형 인터페이스의 추상 메서드를 구현을 직접 전달할 수 있으므로 전체표현식을 인터페이스의 인스턴스로 취급할 수 있다.
    - @FunctionalInterface

- `함수형 인터페이스` 
    - Predicate (T -> boolean)
    
    ```java
    @FunctionalInterface
    public interface Predicate<T> {
        boolean test(T t);
    }
    ```

    - Consumer(T -> void)
     ```java
    @FunctionalInterface
    public interface Consumer<T> {
	void accept(T t);
    }
    ````

    - Supllier( () -> T )
    ```java
    @FunctionalInterface
    public interface Supplier<T> {
        T get();
    }
    ```

    - Function( T -> R)
     ```java
    @FunctionalInterface
    public interface Supplier<T> {
        R apply(T t);
    }
    ```

    - 기본형 특화: 제네릭은 참조형 타입만 지정할 수 있지만, Integer, Long 같이 기본타입에 대한 박싱된 타입을 제네릭을 이용할 수 있지만, 오토박싱으로 인한 변환 비용이 소모된다.
    ```java
    @FunctionalInterface
    public interface IntPredicate {
        boolean test(int value);
    }
    ```

### 스트림
- 스트림 API 이전 컬렉션 API를 이용하여 다양한 로직을 처리했다.
- 스트림 API를 활용하여 다른 방식으로 데이터를 처리한다.
- for-each로 작업을 했다면, Stream-API는 루프를 신경 쓸 필요가 없다.(`내부반복`)
- 멀티스레딩
    - 기존 스레드 API로 멀티스레딩 코드를 구현해서 병렬성을 이용하기 쉽지 않다.
    - 자바 스트림 API는 컬렉션을 처리하면서 발생하는 모호함과 반복적인 코드문제, 멀티코어 활용 두가지를 활용했다(내부적으로 fork-join 방식)
        - `fork-join` : 다른 프로세스 혹은 스레드를 여러개로 쪼개어 새롭게 생성하여 작업을 한 후 실행한 작업들의 결과를 다시 취합하는 방식
- 장점
    - 가비지 변수를 만들지 않는다.
    - 선언형 코드를 구현할 수 있다.
    - 여러 빌딩 블록 연산을 연결해서 복잡한 데이터처리 파이프라인을 만들 수 있다.
    - filter, map, collect 같은 고수준 빌딩 블록으로 이루어져 있으므로 특정 스레딩 모델에 제한하지 않고 자유롭게 어떤상황이든 사용할 수 있다.

### default method
- JAVA 8 이전에는 인터페이스를 구현할 때 메서드를 강제하는 방식이었다면 디폴트 메서드를 이용하여 기존 인터페이스를 구현하는 클래스는 바꾸지 않아도 인터페이스를 변경할 수 있다.
    - ex) List, Collection 인터페이스는 이전에 stream, parallelStream 메서드를 지원하지 않았다. 하지만 JAVA 8 인터페이스에 default 메서드로 제공하여 기존 인터페이스를 쉽게 변경할 수 있었다.


