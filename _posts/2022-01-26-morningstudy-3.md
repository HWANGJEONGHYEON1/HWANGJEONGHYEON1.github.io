---
layout: post
title:  "프록시, 데코레이터 패턴"
date:   2022-01-26 08:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 8번 째

# Proxy


### Proxy
- 클라이언트 . 서버
    - 클라이언트 서버에 필요한 것을 요청하고, 서버는 클라이언트의 요청을 처리를 한다.
- 클라이언트가 요청한 결과를 서버에 직접 요청하는 것이 아니라 어떤 대리자를 통해 간적적으로 서버에 요청할 수 있다.
- 프록시의 역할
    - `대체 기능`: 객체에서 프록시가 되려면, 클라이언트는 서버에게 요청을 한 것인지, 프록시에게 요청한 것인지 조차 몰라야한다. 즉 인터페이스를 사용 => 서버 객체를 프록시 객체로 변경해도 클라이언트 코드를 변경하지 않고 동작할 수 있어야한다.(의존관계 주입)
    - 런타임 의존관계 전 & 후
        - 전) client -> server
        - 후) cilent -> proxy -> server 
    - `접근 제어`: 권한에 따른 접근 차단, 캐싱, 지연로딩
    - `부가기능 추가`: 원래 서버가 제공하는 기능에 부가기능을 수행
- GOF 디자인 패턴
    - 둘 다 프록시를 사용하는 기법이지만, 둘의 의도를 따라 데코레이터와 프록시 패턴으로 구분한다.
    - 프록시: 접근제어
    - 데코레이터 패턴: 새로운 기능 추가가 목적

- 캐시 버전 프록시 패턴

```java

public interface Subject {
    String operation();
}


@Slf4j
public class RealProject implements Subject {
    @Override
    public String operation() {
        log.info("실재 객체 호출");
        sleep(1000);
        return "data";
    }

    private void sleep(int mills) {
        try {
            Thread.sleep(mills);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

@Slf4j
public class CacheProxy implements Subject {

    private Subject target;
    private String cacheValue;

    public CacheProxy(Subject target) {
        this.target = target;
    }

    @Override
    public String operation() {
        log.info("프록시 호출");

        if (cacheValue == null) {
            cacheValue = target.operation();
        }
        return cacheValue;
    }
}

public class ProxyPatternClient {
    private Subject subject;

    public ProxyPatternClient(Subject subject) {
        this.subject = subject;
    }

    public void execute() {
        subject.operation();
    }
}

public class ProxyPatternTest {

    @Test
    void noProxyTest() {
        RealProject realProject = new RealProject();
        ProxyPatternClient client = new ProxyPatternClient(realProject);

        client.execute();
        client.execute();
        client.execute();
        // 3초 수행
    }

    @Test
    void cacheProxyTest() {
        RealProject realProject = new RealProject();
        CacheProxy cacheProxy = new CacheProxy(realProject);
        ProxyPatternClient client = new ProxyPatternClient(cacheProxy);
        client.execute(); // 1번 실행 후 
        client.execute(); // 프록시로 대체 수행
        client.execute(); // 프록시로 대체 수행
        // 1초 수행
    }
}

```
    
- 데코레이터 패턴

```java

public interface Component {
    String operation();
}

@Slf4j
public class DecoratorClient {

    private Component component;

    public DecoratorClient(Component component) {
        this.component = component;
    }

    public void execute() {
        String result = component.operation();
        log.info("result = {} ", result);
    }
}

@Slf4j
public class MessageDecorator implements Component {

    private Component component;

    public MessageDecorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        log.info("messageDecorator 실행");
        String result = component.operation();
        String decoration = "*****" + result + "*****";

        log.info("messageDecorator 적용 전 = {}, 후 = {}", result, decoration);
        return decoration;
    }
}

@Slf4j
public class RealComponent implements Component {
    @Override
    public String operation() {
        log.info("RealComponent 실행");
        return "data";
    }
}

@Slf4j
public class DecoratorTest {

    @Test
    void noDecorator() {
        Component realComponent = new RealComponent();
        DecoratorClient decoratorClient = new DecoratorClient(realComponent);
        decoratorClient.execute();
    }

    @Test
    void decorator1() {
        Component real = new RealComponent();
        Component messageDecorator = new MessageDecorator(real);
        DecoratorClient client = new DecoratorClient(messageDecorator);
        client.execute();
    }

    @Test // 체이닝
    void decorator2() {
        Component real = new RealComponent();
        Component messageDecorator = new MessageDecorator(real);
        Component timeDecorator = new TimeDecorator(messageDecorator);
        DecoratorClient client = new DecoratorClient(timeDecorator);
        client.execute();
    }
}
```

### 패턴 정리
- 프록시를 사용하고 해당 프록시가 접근제어가 목적이라면 `프록시 패턴`, 새로운 기능을 추가하는 것이 목적이라면 `데코레이터패턴`
- intent를 잘 판단하여 패턴을 적용해야한다.