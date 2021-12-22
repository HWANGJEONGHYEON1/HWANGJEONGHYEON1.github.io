---
layout: post
title:  "Modern Java - 2"
date:   2021-12-22 13:00:21 +0900
categories: java
---

> 기본에 충실하자 - 2

# Optional

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



