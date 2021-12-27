---
layout: post
title:  "Modern Java - 6"
date:   2021-12-27 14:20:21 +0900
categories: java
---

> 기본에 충실하자 - 6

# Default method
- 디폴트 메서드는 일반 메서드처럼 바디를 가진다
- 디폴트 메서드를 추가해도 구현한 클래스에 영향을 미치지 않는다.
- 클래스가 같은 시그니처를 갖는 여러 디폴트 메서드를 상속하며 생기는 충돌 문제를 해결하는 규칙이 있다.

### Default 메서드란?
- Java8에서 호환상을 유지하면서 API를 바꿀 수 있도록 새로운 기능인 default method를 제공한다.
- default method는 추상메서드에 해당하지 않는다.

### API 변경
- v1

```java

public interface Resizable extends Drawable {
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
}

```

- v2
    - 요구사항 추가 (setRelativeSize 메서드 추가)

```java
public interface Resizable extends Drawable {
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
  void setRelativeSize(int wFactor, int hFactor);
}
```
    
    - 문제
    - `해당 인터페이스를 구현한 모든 클래스를 수정해야한다.`
    - 해결 : default keyword

```java
public interface Resizable extends Drawable {
  int getWidth();
  int getHeight();
  void setWidth(int width);
  void setHeight(int height);
  void setAbsoluteSize(int width, int height);
  default void setRelativeSize(int wFactor, int hFactor) {
      ...
  }
}
```

### default method 활용 패턴
- Optional Method, Multiple Inheritance of Behavior
- Optional Method
    - 인터페이스를 구현하는 클래스에서 비어있는 메서드를 확인한 적이 있을 것이다. 인터페이스에 정의된 메서드이지만, 실제로는 사용하지 않는 메서드라 빈 메서드를 정의해야한다.
    - default method를 이용하면 이러한 메서드 기본 구현을 제공할 수 있다.

```java

interface Iterator<T> {
  boolean hasNext();
  T next();
  default void remove() {
    throw new UnsupportedOperationException();
  }
}

```

- 동작 다중 상속
    - default method를 이용하면 기존에는 불가능했던 다중상속도 가능
    - 자바에서는 한개의 클래스 상속만 가능하지만 인터페잇느느 여러 개 구현 가능하다.

```java

public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAeccess, Cloneable, Serialiizable {

}

```
    
    - Java8 에서는 인터페이스가 구현을 포함할 수 있으므로 클래스는 여러 인터페이스에서 동작을 상속받을 수 있다. 중복되지 않는 최소한의 인터페이스를 유지한다면 재사용과 조합을 쉽게 할 수 있다.
    - 기능이 중복되지 않는 최소의 인터페이스
    - 기존의 코드를 재사용하며 새로운 기능을 제공할 때 Default Method를 활용하면 된다.


### 해석규칙
- 클래스나 슈퍼클래스에서 정의한 메서드가 default method보다 우선권을 갖는다.

```java

public interface A {
    default int log() {
        log.info("## A");
    }
}

public interface B extends A {
    default int log() {
        log.info("## B");
    }
}


public class C implements B, A {
    public static void main(String... args) {
        new C().log(); // B
    }
}

