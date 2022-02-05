---
layout: post
title:  "리플렉션"
date:   2022-01-26 08:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 9번 째

# 리플렉션
> 클래스나 메서드의 메타정보를 동적으로 획득하고, 코드도 동적으로 호출 할 수 있다.
- 주의할 점: 클래스와 메타정보를 사용해서 어플리케이션을 동적으로 유연하게 할 수 있찌만. 런타임 시점에 동작하므로 컴파일 시점에 오류를 잡을 수 없다.

### 리플렉션 예

```java
    // 코드가 중복된다.
    @Test
    void reflection() {
        Hello target = new Hello();
        log.info("start");
        String result1 = target.callA();
        log.info("result1 = {}", result1);

        log.info("start");
        String result2 = target.callB();
        log.info("result2 = {}", result2);
    }

    // 중복되는 부분을 리팩토링할 수 있을것으로 보인다.
    @Test
    void reflection1() throws Exception {
        // class info
        Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

        Hello target = new Hello();

        Method methodCallA = classHello.getMethod("callA");
        Object result1 = methodCallA.invoke(target);
        log.info("result1 = {}", result1);

        Method methodCallB = classHello.getMethod("callB");
        Object result2 = methodCallB.invoke(target);
        log.info("result2 = {}", result2);

    }
```

- 리팩토링 버전
    - 기존에는 메서드 이름을 직접 호출했찌만, 이제는 method 라는 메타정보를 통해서 호출할 메서드 정보가 동적으로 제공된다.

```java
    @Test
    void reflection2() throws Exception {
        Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

        Hello target = new Hello();


        Method methodCallA = classHello.getMethod("callA");
        dynamicCall(methodCallA, target);

        Method methodCallB = classHello.getMethod("callB");
        dynamicCall(methodCallB, target);
    }

    private void dynamicCall(Method method, Object target) throws Exception {
        log.info("start");
        Object result = method.invoke(target);
        log.info("result = {}", result);
    }
```

### JDK 동적 프록시
- 동적프록시 기술을 사용하면 개발자가 직접 프록시 클래스를 만들지 않아도 된다.
- 이름 그대로 프록시 객체를 동적으로 런타임 개발자 대신 만들어준다. 동적 프록시에 원하는 실행 로직을 지정할 수 있다.

```java
    @Test
    void dynamicA() {
        AInterface target = new AImpl();
        TimeInvocationHandler handler = new TimeInvocationHandler(target);

        AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);

        proxy.call();
        log.info("targetClass = {}", target.getClass());
        log.info("proxy = {}", proxy.getClass());
    }

```

### CGLIB
- 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리
- 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들 수 있다.

```java
    @Test
    void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("target class : {}", target.getClass());
        log.info("proxy class : {}", proxy.getClass());
        proxy.call();
    }
```

> JDK 동적프록시는 인터페이스를 구현해서 프록시를 만든다. CGLIB는 구체클래스를 상속 해서 프록시를 만든다.

