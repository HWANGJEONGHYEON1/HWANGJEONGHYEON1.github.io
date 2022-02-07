---
layout: post
title:  "리플렉션"
date:   2022-01-26 08:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 9번 째

# 리플렉션
> 클래스나 메서드의 `메타정보`를 동적으로 획득하고, 코드도 동적으로 호출 할 수 있다.
- 주의할 점: 클래스와 `메타정보`를 사용해서 어플리케이션을 동적으로 유연하게 할 수 있찌만. 런타임 시점에 동작하므로 컴파일 시점에 오류를 잡을 수 없다.

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
        log.info("targetClass = {}", target.getClass()); //class hello.proxy.jdkdynamic.code.AImpl
        log.info("proxy = {}", proxy.getClass()); //class com.sun.proxy.$Proxy9
    }

```

### CGLIB
- 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리
- 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 만들 수 있다.
- 제약
    - 기본생성자가 필요하다
    - final 키워드가 있으면 상속 불가능


```java
    @Test
    void cglib() {
        ConcreteService target = new ConcreteService();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteService.class);
        enhancer.setCallback(new TimeMethodInterceptor(target));
        ConcreteService proxy = (ConcreteService) enhancer.create();
        log.info("target class : {}", target.getClass()); //class hello.proxy.common.service.ConcreteService
        log.info("proxy class : {}", proxy.getClass()); // class hello.proxy.common.service.ConcreteService$$EnhancerByCGLIB$$25d6b0e3
        proxy.call();
    }
```

> JDK 동적프록시는 인터페이스를 구현해서 프록시를 만든다. CGLIB는 구체클래스를 상속 해서 프록시를 만든다.

### 프록시 팩토리
- 이 전 문제점
    - 인터페이스가 있을 때는 JDK 동적프록시 적용, 아니면 CGLIB를 적용
    - 두 기술을 함께 사용할 때 부가기능을 제공하기 위해 JDK 동적 프록시가 제공하는 InvocationalHandler 와 CGLIB가 제공하는 MethodInterceptor를 중복으로 각각 만들어야한다.
    - 특정 조건에 맞을 때 프록시 로직을 적용하는 기능이 있다면 어떻게 ?
- 스프링은 유사한 기술이 있을 때 그것들을 통합해서 일관성있게 제공하고, 더욱 편리하게 추상화 된 기술을 제공한다.
- 문제점 해결
    - `프록시 팩토리`
        - 인터페이스가 있으면 JDK 동적 프록시 사용
        - 구체클래스만 있으면 CGLIB를 사용
        - 의존 관계
            - client -> proxyFactory -> jdk or cglib
- 프록시가 제공하는 부가 기능 로직을 `Advice`라고 한다.
- 정리
    - CGLIB, JDK 동적 프록시에 의존하지 않고 동적 프록시를 생성할 수 있다.
    - 스프링 부트는 AOP를 적용할 때 기본적으로 `proxyTargetClass=true`로 설정해서 사용, 인터페이스가 있어도 항상 CGLIB를 사용해서 구체 클래스를 기반으로 프록시를 사용한다.
- 코드

```java
    
    @Slf4j
    public class TimeAdvice implements MethodInterceptor {
        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {

            log.info("TimeAdvice 실행");
            long startTime = System.currentTimeMillis();
            Object result = invocation.proceed(); // target 클래스를 호출하고 그 결과를 받는다.
            long endTime = System.currentTimeMillis();
            long resultTime = endTime - startTime;
            log.info("TimeAdvice 종료 : resultTime: {}", resultTime);
            return result;
        }
    }


    @Test
    @DisplayName("인터페이스가 있으면 JDK 동적 프록시 사용")
    void interfaceProxy() {
        ServiceInterface target = new ServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();
        log.info("target : {}", target.getClass()); // target : class hello.proxy.common.service.ServiceImpl
        log.info("proxy : {}", proxy.getClass()); //  proxy : class com.sun.proxy.$Proxy10
        proxy.find();

        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isFalse();
    }

    @Test
    @DisplayName("구체클래스가 있으면 cglib 프록시 사용")
    void concreteProxy() {
        ConcreteService target = new ConcreteService();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.addAdvice(new TimeAdvice());
        ConcreteService proxy = (ConcreteService) proxyFactory.getProxy();
        log.info("target : {}", target.getClass()); //target : class hello.proxy.common.service.ConcreteService
        log.info("proxy : {}", proxy.getClass()); // class hello.proxy.common.service.ConcreteService$$EnhancerBySpringCGLIB$$44199858
        proxy.call();

        Assertions.assertThat(AopUtils.isAopProxy(proxy)).isTrue();
        Assertions.assertThat(AopUtils.isJdkDynamicProxy(proxy)).isFalse();
        Assertions.assertThat(AopUtils.isCglibProxy(proxy)).isTrue();
    }

```

### 포인트 컷, 어드바이스, 어드바이저
* 포인트 컷: 어디에 부가 기능을 적용할지, 어디에 부가기능을 적용하지 않을지 판단하는 필터링 로직, 어떤 포인트에 기능을 적용할지 안할지
* 어드바이스: 프록시가 호출하는 부가 기능. 프록시 로직
* 어드바이저: 하나의 포인트컷과 하나의 어드바이스를 가지고 있는것 => 포인트컷1 + 어드바이스1
* 역할과 책임
    * 포인트 컷은 대상여부를 확인하는 필터 역할만 담당
    * 어드바이스는 깔끔하게 부가 기능 로직만 담당
    * 둘을 합치면 어드바이저


### 프록시 중요점
- 스프링 AOP를 적용할 때, 최적화를 진행해서 지금처럼 프록시는 하나만 만들고, 하나의 프록시에 여러 어드바이저를 사용
- 하나의 target에 여러 AOP가 적용되더라도, target마다 하나의 프록시만 생성된다.