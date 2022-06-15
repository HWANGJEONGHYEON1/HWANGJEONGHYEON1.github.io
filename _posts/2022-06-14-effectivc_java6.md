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
