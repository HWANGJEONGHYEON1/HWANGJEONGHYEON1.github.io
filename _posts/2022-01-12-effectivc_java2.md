---
layout: post
title:  "Effectivce Java - 2"
date:   2022-01-12 09:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 2

# 객체 생성과 파괴

### item6) 불필요한 객체 생성을 피해라

```java

String s = new String("aa"); // 안좋은 방법
String s = "aa"; // 문자열 리터럴을 사용하는 모든 코드가 같은 객체임을 보장
```

- 밑의 코드는 새로운 인스턴스를 매번 만드는 대신 하나의 String 인스턴스를 사용한다.
    - 같은 가상 머신안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체임을 보장된다.
- 생성 비용이 비싼 객체라면 캐싱을 고려하자
    - 정규표현식에 해당되는 Pattern은 생성비용이 높다.
    - 인스턴스를 클래스 초기화 과정에서 직접 생성해 캐싱해두고 나중에 해당 메서드가 호출될 때마다 이 인스턴스를 재사용하면 성능이 전보다 나아진다.
- 객체 생성은 비싸니까 피해야한다 라는 의미가 아니다
    - JVM입장에서는 작은 객체를 생성하고 회수하는 일은 부담이 되지 않는다.
        - 프로그램의 명확성, 간결성, 기능을 위해서 좋은 일

```java
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++ ) {
        sum += i;
    }

    return sum; // 실행시간 6초 => Long sum to long sum 실행시간 0.59초

```

- 박싱된 기본타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱에 주의해야한다.


### itme7) 다 쓴 객체 참조를 해제하라

- 가비지 컬렉션 언어에서는 메모리 누수를 찾기가 힘들다. 객체 참조 하나를 살려두면 가비 컬렉터는 그 객체 뿐 아니라 그 객체가 참조하는 모든 객체를 회수하지 못한다.
- 해법 : 다 썻다면 null 처리를 통해 참조 해제를 하면 된다.

```java

pbulic Object pop() {
    if (size == 0) {
        throw new EmptyStacException();
    }
    return elements[--size];
}

pbulic Object pop() {
    if (size == 0) {
        throw new EmptyStacException();
    }
    Object result = elements[--size];
    elements[--size] = null; // 참조 해제
    return result;
}

```

- 캐시 역시 메모리 누수를 일으키는 주범이다
    - 객체참조를 캐시에 넣고 까먹는 경우가 있다
        - 해법: WeakHashMap: 다 쓴 엔트리는 그 즉시 자동으로 제거된다.
        - 이러한 상황에 유의하여 사용해야한다.
    - 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수작업으로 수행하는 방법
        - LinkedHashMap은 removeEldestEntry 메서드 사용한다.

### item8) finalizer와 cleaner 사용을 피하라
- finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
- cleaner가 대안으로 나왔지만, 여전히 예측할 수 없고 느라고, 일반적으로 불필요하다.
- finalizer cleaner는 제때 실행되어야하는 작업은 절대 할 수가 없다.


### item9) try-finally 보다는 try-with-resources를 사용하라
- 자원이 제대로 닫힘을 보장하는 수단으로는 try-catch가 쓰였다.

```java
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finallay {
        br.close();
    }
```

- try - resource - with
    - 꼭 회수해야하는 자원이 있을 때 사용
    - 코드는 짧고 분명해진다.

```java

    try(BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
```

# 모든 객체의 공통 메서드
> Equals, hashcode, toString은 모두 재정의 염두에 두고 설계된 것

### item10) equals는 일반 규약을 지켜 재정의하라
- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 `논리적 동치성`을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equlas가 하위 클래스에도 딱 들어 맞는다.
- 클래스가 private 이거나 pakage-private이고 equlas 메서드 호출할 일이 없다.
- 재정의 해야할 때
    - 객체 실별성이 아니라 논리적 동치성을 확인해야하는대, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
- equlas메서드는 동치관계를 구현하며, 다음을 만족
    - 반사성 : null이 아닌 모든 참조값 x에 대하, x.equals(x) = true
    - 대칭성 : null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true
    - 추이성 : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true면 y.equals(x)도 true, x.equals(z)이면 z.equals(y)도 true
    - 일관성 : null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)는 항상 true 또는 false를 밥환한다.
    - null-아님 : null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false다.
- 동치관계 : 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산(서로 자원을 바꿔도 모두 같은 부류여야한다.)
- 반사성 : 객체는 자기 자신과 같아야한다. 
- 대치성 : 두 객체는 서로에 대한 동치 여부에 똑같이 답해야한다.
- 추이성 위반
    - 두 클래스가 있을 때, equals 메서드를 쓴다면, ColorPoint 클래스를 비교하게 되었을 때 color를 무시하고 비교를 하게된다.
    - point.equals()

```java

public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        Point p = (Point) o;
        return x== p.x && y == p.y;
    }
}


public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        
        return super.equals(o) && ((ColorPoint) o).color == color;
    }

}
        // 대칭성 위반! 
        Point p = new Point(1, 2);
        ColorPoint colorPoint = new ColorPoint(1, 2, Color.RED);
        System.out.println(p.equals(colorPoint)); // true
        System.out.println(colorPoint.equals(p)); // false

        // 추이성 위반!
        ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
        Point p2 = new Point(1, 2);
        ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);

        System.out.println(p1.equals(p2)); // t
        System.out.println(p2.equals(p3)); // f
        System.out.println(p1.equals(p3)); // t
```

- equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.
- 양질의 equals 구현 방법
    - == 연산자를 이용하여 입력이 자기 자신이 참조인지 확인.
    - instanceof 연산자로 입력이 올바른 타입인지 확인.
    - 입력을 올바른 타입으로 형변환
    - 입력 객체와 자기 자신의 대응되는 `핵심` 필드들이 모두 일하는지 하나씩 검사
    - equals를 다 구현했다면, 대칭적, 추이성, 일관성인지 확인해야한다.


```java
public class PhoneNumber {
    
    private final short areaCode, prefix, lineNumber;

    public PhoneNumber(short areaCode, short prefix, short lineNumber) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNumber = rangeCheck(lineNumber, 9999, "가입자번호");
    }

    private short rangeCheck(short val, int max, String arg) {
        if (areaCode < val || areaCode > max) {
            throw new IllegalArgumentException("값이 정상아님");
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode && prefix == that.prefix && lineNumber == that.lineNumber;
    }

    @Override
    public int hashCode() {
        return Objects.hash(areaCode, prefix, lineNumber);
    }
}
```

- equals를 재정의할 땐, hashCode도 반드시 재정의해라
- 복잡하게 해결하려고 하지말자
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자
    - public boolean equals(AnotherClass class){}


### item11) equals를 재정의하려거든 hashcode도 재정의하라
- equals를 재정의한 클래스에서 모두 hashcode도 재정의해야한다.
    - 규약을 어긴다면 hashcode 일반규약을 어기게되어 클래스의 인스턴스 HashMap, HashSet 같은 컬렉션의 원소로 사용할 때 문제가 된다.
- equal 비교에 사용되는 정보가 변경되지 않으면, 애플리케이션이 실행되는 동안 그 객체의 hashcode 메서드는 몇번을 호출해도 일관되게 항상 같은 값을 반환해야한다.
- equals가 두 객체를 같다고 판단했다면, 두 객체의 hashcode는 똑같은 값을 반환해야한다.
- euqals가 두객체를 다르다고 판단했더라도, 두 객체의 hashcode가 서로 다른 값을 반환할 필요는 없다. 단, 다른 값을 반환해야 해시테이블의 성능이 좋아진다.
- hashCode 재정의를 잘못햇을 때 크게 문제가 되는 조하응 두번째, 논리적으로 같은 객체는 같은 해시코드를 반환해야한다.

```java
Map<PhoneNumber, String> m = new HashMap<>();

m.put(new PhoneNumber(1,2,3), "jin");

sysout(m.get(new PhoneNumber(1,2,3))) // null

@Override
public int hashCode() {return 42;} // 재정의해줌으로써 jin을 반환하지만 옯바르지 않다.

```

- public int hashCode() {return 42;} 
    - 모든 해시값이 42이므로 같은 버킷 하나에 담겨 LinkedList처럼 동작한다.
    - hashmap의 O(1) -> O(N)으로 바뀔 수 있다.
- 이상적인 해시함수는 주어진 인스턴스를 32비트 정수범위에 균일하게 분배해야한다.

```java

public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short(prefix);
    result = 31 * result + Short(lineNum);
    return result;
}
```