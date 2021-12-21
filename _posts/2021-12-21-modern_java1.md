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

### default method
- JAVA 8 이전에는 인터페이스를 구현할 때 메서드를 강제하는 방식이었다면 디폴트 메서드를 이용하여 기존 인터페이스를 구현하는 클래스는 바꾸지 않아도 인터페이스를 변경할 수 있다.
    - ex) List, Collection 인터페이스는 이전에 stream, parallelStream 메서드를 지원하지 않았다. 하지만 JAVA 8 인터페이스에 default 메서드로 제공하여 기존 인터페이스를 쉽게 변경할 수 있었다.



### Optional - 값이 없는 상황을 처리
- NPE 줄이기
    - NULL 때문에 발생하는 문제
        - NPE는 흔한 에러
        - 코드가 더러워진다,
        - NULL은 의미가 없다(값이 없다라는 의미도 적절하지 않다)
        - 자바 철학에 위배
        - 형식 시스템에 구멍을만든다 => 모든 레퍼런스는 NULL이 할당 가능하다. NULL이 점점 확장됬을 때 어떤 의미의 NULL인지 알 수 없다.
- Optional 클래스
    - 하스켈과 스칼라의 영향을 받아 클래스가 제공된다.
    - Optional은 선택형 값을 캡슐화하는 클래스
    - 값이 있으면 Optional 클래스는 값을 감싼다. 없으면 Optional.empty() 메서드로 Optional을 반환.
    - empty()는 싱글턴인스턴스를 반환하는 `정적팩토리메서드`이다.
    - Optional이 등장하면 unWrap해서 값이 없을 수 있는 상황에 적절하게 대응하도록 강제하는 효과가 있다.

```java

Optional<Animal> optionalAnimal = Optional.empty(); // 빈 Optional

Optional<Animal> optionalAnimal = Optional.of(animal); // null이 아닌 값으로 Optional 

Optional<Animal> optionalAnimal = Optional.ofNullable(animal); // null 값을 Optional로 만들기, animal이 null이면 Optional 빈객체가 반환

```

- 스트림의 map과 비슷하게 제공된 함수를 적용할 수 있다. 

```java
Optional<Animal> optionalAnimal = Optional.of(animal); 
Optional<String> animalName = optionalAnimal.map(Animal::getName);
```
 
- 두번 감싸진 Optional로 받고 싶지 않을 때는 flatMap 사용
- Optional 클래스는 필드형식으로 사용할 것을 가정하지 않았기 때문에 Serializable 인터페이스를 구현하지 않았다. 따라서 직렬화 모델을 사용할 경우 문제가 생길 수 있다.
- 자바 9에서는 Optinal을 포함하는 스트림을 쉽게 처리할 수 있도록 stream()메서드를 추가했다.

```java
Stream<Optional<Animal>> streamOptionVariable = ..;
Set<Animal> result = stream.filter(Optional::isPresent)
                        .map(Optional::get)
                        .collect(Collectors::toSet);

```


### optional 메서드
- get: 값을 읽을 순 있지만 안전하지 않다. -> 값이 없으면 NoSuchElementException 예외가 터진다.
- ifPresent(Consumer<? super T> consumer): 값이 존재할 때 인수로 넘겨진 동작을 실행할 수 있다. 값이 없으면 아무일도 일어나지 않음
- orElse: Optional이 값을 포함하지 않을 때 기본값을 제공할 수 있다.
- orElseGet(Supplier<? extends T> suplier): orElse 메서드에 대응하는 게으른 버전의 메서드. Optional에 값이 없을 때만 Suplier가 실행
- orElseThrow(Supplier<? extends T> exSuplier): 값이 존재하지 않는다면, 예외를 발생시킨다.

### Optional을 사용해야할 때
- 잠재적으로 NULL이 될 수 있는 대상을 Optional로 감싸기
- 예외와 Optional을 반환해야할 때 orElse* 를 사용하자
