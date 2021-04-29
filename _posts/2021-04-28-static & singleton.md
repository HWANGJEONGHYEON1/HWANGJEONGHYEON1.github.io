---
layout: post
title:  "static & singleton"
date:   2021-04-29 10:33
categories: java
tags: [java]
---

### static
* static 이란
    - `프로그램 실행 시 한번 선언되면 메모리에 올라가 종료될 때까지 메모리 상에 남아있는다.`

* 특징
    - 정적메소드에서는 정적필드만 사용할 수 있다.
    - `this` 키워드 사용 불가.
    - 메소드 오버라이딩 사용 불가


* 장점
    - 편리함 
        - 별도의 객체선언 없이 사용 가능. (부가적으로 무의미한 객체생성을 막고 메모리를 아낄수도 있다.)
        - 공유하기 쉽고 쉽게 호출
    
* 단점
    - 메모리 과도
        - 스태틱을 남발하게 된다면 static 당 메모리를 차지해 memoryOverFlow 발생

* static block

```
    static {
        ...
    }
```
- 클래스가 로드될 때 자동으로 실행
- 클래스내에 여러개 선언되어도 상관없다.
- 인스턴스 필드, 메서드는 사용불가 (객체로 아직 생성되지 않음)

## 스태틱은 되도록 사용하지 않도록 권장
 * 싱글톤 대체
    - 클래스가 최초 한번만 메모리에 로딩하고, 그 메모리에 객체를 생성하여 사용하는 디자인 페턴
 ```java
    public class Singleton {
        public static Singleton singleton = new Test();

        private Singleton(){}

        public static Singleton getInstance() {
            return singleton;
        }
    }

    public Class Test {
        Singleton a;
        Singleton b;
        
        public void method() {
            a = Singleton.getInstance();
            b = Singleton.getInstance();
            
            if (a == b) {
                System.out.print("같은 객체");
            } else {
                System.out.print("다른 객체");
            }
        }
    }
 ```
