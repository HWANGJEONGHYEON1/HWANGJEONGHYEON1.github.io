---
layout: post
title:  "Effectivce Java - 6"
date:   2022-06-14 16:20:21 +0900
categories: java
---

> #2 기본에 충실하자 

# 메소드

## item49) 메개변수가 유효한지 검사하라
- 메서드 몸체가 실행되기전에 매개변수를 확인한다면 잘못된 값이 넘어왔을 때 즉각적이고 깔끔하게 예외를 던질 수 있다.
- public, protected 메서드는 매게변수 값이 잘못 됬을 때 던지는 예외를 문서화해야한다.
- ex
```java
    /**
    * (현재 값 mod m ) 값을 반환
    * 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
    * @param m 계수(양수여야한다.)
    * @return 현재 값 mod m
    * @throws ArthmetricException m이 0보다 작거나 같으면 발생
    */
    public BigInteger mod(BigInteger m) {
        if (m.signum() <= 0) {
            throw new ArthmetricException("계수는 양수여야합니다. = " + m);
        }
    }
```

- 현재 메서드는 m이 null이면 NPE를 던지는대 설명은 없다. 
- 이유는 설명을 BigInteger 클래스 수준으로 설명했기 때문이다.
- public 이 아닌 메서드라면 단언문(assert)를 사용해 매개변수 유효성을 검증할 수 있다.

```java
    assert a !- null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
```
- 이 단언문의 핵심은 자신인 단언한 조건이 무조건 참이라고 선언하는 것.
    1. 실패하면 AssertionsError를 던진다.
    2. 런타임에 아무런 효과도 성능 저하도 없다


## item50) 적시에 방어적 복사본을 만들라
- 자바는 안전한 언어이지만, 클라이언트가 불변식을 깨뜨리려 혈안이 되어 있다고 생각하고 프로그래밍을 해야한다.

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        if (start.comparedTo(end) > 0) {
            throw new Exception("error");
        }

        this.start = start;
        this.end = end;
    }
    ...
}
```

- 위의 코드는 불변적으로 보이지만, 사실 Date가 가변이다.
```java
    Date start = new Date();
    Date end = new Date();
    Period p = new Priod(start, end);
    end.setYear(78); // 문제

```
- 자바 8 이후로는 해결할 수 있다.
    - Date 대신 Instance를 사용하면 된다
        - LocalDateTime, ZonedDateTime
- Date는 낡은 API이므로 새로운 코드를 작성할 때는 사용하면 안된다.
- 외부 공격으로 Period 내부를 보호하려면 생성자에서 받은 다음 가변 매게변수를 각각을 방어적으로 복사해야한다.
    - Period 안에서는 원본이 아닌 복사본을 사용한다.

```java
    public Period(Date start, Date end) {
        if (start.comparedTo(end) > 0) {
            throw new Exception("error");
        }

        this.start = start.getTime();
        this.end = end.getTime();
    }

```
- 매개변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.
- 생성자를 수정하면 앞서의 공격은 막아낼 수 있지만, Period 인스턴스는 아직도 변경이 가능하다.
    - p.end().setYear(); // 이 부분
    - 막는 방법은 가변  필드의 방어적 복사본을 반환하면 된다.
    ```java
        public Date end() {
            return new Date(end.getTime());
        }
    ```
- 클래스가 클라이언트로부터 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사해야한다.
- 복사비용이 너무 크거나 클라이언트가 그 요소를 잘못 수정할 일이 없음을 신뢰한다면 책임이 클라이언트에 있음을 문서에 명시하자.

## item51) 메서드 시그니처를 신중히 설계하라
- 메서드이름을 신중히 짓자.
    - 개발자 커뮤니티에서 자주 사용하는 것으로
- 편의 메서드를 많이 만들지 말자.
    - 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야한다.
    - 확신이 서지 않으면 만들지말자.
- 매개변수 목록은 짧게 유지하자
    - 4개 이하로 만들자
    - 같은 타입의 매개변수가 여러 개가 연달아 나오는 경우가 특히 해롭다
- 매개변수 목록을 짧게 줄여주는 기술 3가지
    - 여러 메서드로 쪼갠다.
    - 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다.
    - 위 두가지를 혼합한 것으로, 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다.
- 매개변수의 타입으로는 클래스보다 인터페이스가 낫다.
- Boolean 보다는 원소 두개짜리 열거타입이 낫다.

## item52) 다중 정의는 신중히 사용하라
- 재정의한 메서드는 동적으로 선택이되고, 다중정의한 매서드는 정적을 선택된다.
- 다중 정의된 메서드 사이에서는 객체의 런타임 타입은 전혀 중요하지 않다.
- 선택은 컴파일타임에, 오직 매개변수의 컴파일타임 타입에 이뤄진다.

# item53) 가변인수를 신중히 사용하라
- 인수 개수가 일정하지 않은 메서드를 정의해야한면, 가변인수가 반드시 필요하다.
- 메서드를 정의할 때 필수 매개변수는 가변인수 앞에두고, 가변인수를 사용할 때는 성능문제까직 고려하자


## item54) null이 아닌, 빈 컬렉션이나 베열을 반환하라

```java
private List<Cheese> getChesses() {
    return cheessesInStock.isEmpty() ? null : new ArrayList<>(cheessesInStock);
}

...

if (cheese != null) && cheeses.contains(Cheese.STILION)) {
    ...
}

```

- 컬렉션이나 배열같은 컨테이너가 비었을 때 null을 반환하는 메서드를 사용할 때면 항시 이와같은 방어코드를 넣어야한다.
- null을 반환하면 두가지 틀린 주장이 있다.
    - 성능 분석 결과 이 할당이 성능저하의 주범이라고 확인되지 않았다 => 성능문제는 아니다.
    - 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.
- Collections.empty*()을 사용하자
- 배열이라면 new CheesseArray[0]을 사용하면 된다.

