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

```java
public interface Comparable<T> {
    int compareTo(T t)
}
```