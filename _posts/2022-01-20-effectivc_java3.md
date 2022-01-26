---
layout: post
title:  "Effectivce Java - 3"
date:   2022-01-20 09:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 3

# 모든 객체의 공통 메서드

### item12) toString을 항상 재정의하라
- toString을 잘 구현한 클래스는 사용하기에 좋고, 디버깅이 쉽다.
- toString은 해당 객체에 관해 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야한다.

### item14) Comparable을 구현할지 고려하라
- 메서드 compareTo는 단순 동치성 비교
- 알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성하면 반드시 Comparable을 구현하자.
- 순서를 고려해야하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여 정렬, 검색, 비교 기능을 제공하는 컬렉션과 어우지도록 해야한다.
- compareTo 메서드에서 필드 값을 비교할 때 <. > 연산자는 쓰지 않아야한다.
- 박싱된 기본타입 클래스가 제공하는 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자

```java
public interface Comparable<T> {
    int compareTo(T t)
}
```

# 클래스와 인터페이스

### item15) 클래스와 멤버의 접근 권한을 최소화하라

- 설계가 잘된 큰 차이는 내부 구현 정보를 얼마나 잘 숨겼느냐에 따라 달렸다.
    - 이를 정보은닉, 캡슐화라고 한다.
- 정보은닉 장점
    - 시스템 개발속도를 높인다. 여러 컴포넌트를 병렬로 개발 할 수 있다.
    - 시스템 관리 비용을 낮춘다.
    - 성능 최적화에 도움이 된다.
    - 소프트웨어의 재사용성을 높인다.
- 정보은닉의 핵심
    - 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야한다.
    - public 클래스의 인스턴스 필드는 되도록 public이 아니어야한다.
        - 이런 경우 일반적으로 스레드 안전하지 않다.
    - public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안된다.
        - public static final Things[] value= {..};
        - 해결책

        ```java
            public static final Things[] PRIVATE_VALUE = {..};
            private static final List<Things> VALUES = Collections.unmodifieableList(Arrays.asList(PRIVATE_VALUE)) // 1
            public static final Thing[] values() {
                return PRIVATE_VALUE.clone();
            }
        ```

    - private: 멤버를 선언한 톱 레벨 클래스에서만 접근가능
    - protected: 이 멤버를 선언한 하위클래스에서 접근가능
    - public: 모든 곳에서 접근가능

### item16) public 클래스에서는 public 릴드가 아닌 접근자 메서드를 이용하라
- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든지 바꿀 수 있는 유연성을 얻을 수 있다.


### item17) 변경가능성을 최소화하라
- 불변 클래스: 인스턴스의 내부 값을 수정할 수 없는 클래스
    - 가변 클래스보다 설계하고 구현하기 쉬우며, 오류가 생길 여지가 적고 안전하다.
- 불변 만드는 방법
    - 객체의 상태를 변경하는 메서드를 제공하지 않는다.
    - 클래스를 확장할 수 없도록 한다.
        - final 또는 다른 방법
    - 모든 필드를 final로 선언한다.
        - 시스템이 강제하는 수단을 이용해 설계자의 명확한 의도를 나타낸다.
    - 모든 필드를 private로 선언한다.
    - 자신 외에는 내부 의 가변 컴포넌트에 접근할 수 없도록 한다.
- 불변객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- 불변 객체는 근본적으로 스레드 세이프하여 따로 동기화할 필요 없다.
- 불변객체는 안심하고 공유할 수 있다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
    - 불변객체는 맵의 키와 집합의 원소로 쓰이기에 안성맞춤
- 불변 객체는 그 자체로 실패 원자성을 제공한다.
    - 상태가 절대 변하지 않으니 불일치 상태에 빠질 가능성이 없다.
- 단점
    - 값이 다르다면 반드시 독립된 객체로 만들어야한다.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.

### item18) 상속보다는 합성을 이용하라
- 메서드 호출과 달리 쌍속은 캡슐화를 깨뜨린다.
    - 상위 클래스가 어떻게 구현하느냐에 따라서 하위 클래스의 동작에 이상이 있을 수 있다.

```java
public class InstrumentHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getCount() {
        return addCount;
    }
}

    public static void main(String[] args) {
        InstrumentHashSet<String> s = new InstrumentHashSet<>();
        s.addAll(List.of("틱", "틱틱", "펑")); 
        System.out.println(s.getCount());// 3예상
    }

```

- 3이 출력될 것  같았지만 6이나온다.
- 원인
    - HashSet의 addAll은 각 원소를 add 메서드를 통해 호출하여 addCount가 중복해서 더해져 최종 갑으로 6이 나온다.
```java
    // HashSet의 addAll
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c)
            if (add(e))
                modified = true;
        return modified;
    }

```

- 해결 => 컴포지션

```java
public class InstrumentedHashSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "틱틱", "펑"));
        System.out.println(s.getCount());// 3
    }
}
```



