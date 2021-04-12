---
layout: post
title:  "JAVA final"
date:   2021-04-01 19:22:21 +0900
categories: java
---
# final

## final에 대해 정리하게 된 이유

- 스프링 공부를 하던 도중 김영한(스프링 강의)님께서 변수들을 final로 해놓은 것들을 보았고 왜 다 final을 붙일까 생각이 들었다.


> ```final```
> 
> the final keyword is used in several contexts to define an entity that can only be assigned once. 
>
>Immutable object - 객체를 공유하여 스레드 세이프하다.
>
>Read-Only
>
>Side-effect가 적다.
>

_클래스_ 
```java
public final class App{}
````
- 상속이 불가능하다.
- 생성자가 초기화 되어있는 경우 초기화 불가능

_메서드_
```java
public final void test{}
````
- 오버라이드를 할 수 없다.

_변수_
```java
final String name;
```
- 값을 재할당 할 수 없다.
- 기본 생성자를 사용할 수 없다.


```final class``` 로 상속이 불가능하도록 생성하고 생성 시 전달받은 인자를 수정할 수 없게 ```Immutable``` 하게 만들어준다.

```가독성이 좋다``` > 수식을 이해는대 좋다. (아직 잘 모르겠다.)


### 엘레강트 오브젝트 책
- 책에서는 모드 클래스와 변수를 final로 지정해서 해야한다.
- 상속이 객체들의 관계를 복잡하게 만들기 때문에 상속이 좋지않다 -> final로 제한한다면 가능성을 없앨수 있다.

참조 [Link](https://makemethink.tistory.com/184).

