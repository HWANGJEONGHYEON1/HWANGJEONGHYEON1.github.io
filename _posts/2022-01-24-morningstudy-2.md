---
layout: post
title:  "전략패턴"
date:   2022-01-24 08:20:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근)

# 강의 정리(스프링) - 5번 째


## 전략패턴
- 템플릿 메서드패턴은 변하지 않는 부분을 템플릿을 두고, 상속을 받아 문제 해결
- 전략패턴은 변하지 않는 부분을 context라는 곳에 두고, Stretagy 라는 인터페이스를 만들고 해당 인터페이스를 구현하도록 해서 문제 해결
    - 상속이 아니라 `위임`
- Gof 디자인 패턴
    - 알고리즘 제품군을 정의하고 각각을 캡슐화 하여 상호 교환 가능하게 만든다. 전략을 사용하면 알고리즘을 사용하는 클리아언트와 독립적으로 알고리즘을 변경할 수 있다.


### 코드
- Context에 원하는 전략을 구현체에 주입
- 클라이언트는 Context를 실행
- context는 로직을 실행
- context는 전략을 주입받아 로직을 실행

```java
public interface Strategy {
    void call();
}

@Slf4j
public class StrategyLogic1 implements Strategy {
    @Override
    public void call() {
        log.info("비지니스로 직 1 실행 ");
    }
}


@Slf4j
public class StrategyLogic2 implements Strategy {
    @Override
    public void call() {
        log.info("비지니스로 직 2 실행 ");
    }
}

@Slf4j
public class ContextV1 {

    private Strategy strategy;

    public ContextV1(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute() {
        long startTime = System.currentTimeMillis();

        strategy.call();
        long endTime = System.currentTimeMillis();
        log.info("resultTime {}", endTime - startTime);
    }
}

@Slf4j
public class ContextV1Test {
    @Test
    void strategyV0() {
        logic1();
        logic2();
    }

    private void logic1() {
        long startTime = System.currentTimeMillis();
        log.info("# business execution 1");

        long endTime = System.currentTimeMillis();
        log.info("resultTime {}", endTime - startTime);

    }

    private void logic2() {
        long startTime = System.currentTimeMillis();
        log.info("# business execution 2");

        long endTime = System.currentTimeMillis();
        log.info("resultTime {}", endTime - startTime);

    }

    @Test
    void strategyV1() {
        StrategyLogic1 strategyLogic1 = new StrategyLogic1();
        ContextV1 contextV1 = new ContextV1(strategyLogic1);
        contextV1.execute();

        StrategyLogic2 strategyLogic2 = new StrategyLogic2();
        ContextV1 contextV2 = new ContextV1(strategyLogic2);
        contextV2.execute();
    }
}


```

### 선 조립, 후 실행
- context와 전략을 실행하기 전에 원하는 모양으로 조립해두고, 그 다음에 context를 실행하는 선 조립, 후 실행 방식에서 유용하다.
- 스프링 어플리케이션 로딩 시점에 의존 관계 주입을 통해 필요한 의존관계를 모두 맺어두고 난 다음에 실제 요청을 처리해주는것과 같다.
- 단점
    - 조립 한 이후에 전략을 변경하기 어렵다.
- 단점 보완 방법 
    - 전략을 조립 후 실행이 아니라, 전략을 실행할 때마다 인수로 전달한다(메서드 파라미터 사용)
    - 이전 방식과는 비교하여 좀 더 유연하다.

```java
@Test
    void strategyV2() {
        ContextV1 contextV1 = new ContextV1(() -> log.info("비지니스 실행"));
        contextV1.execute();

        ContextV1 contextV2 = new ContextV1(() -> log.info("비지니스 실행"));
        contextV2.execute();
    }
```

## 템플릿 콜백 패턴
- 다른 코드의 인수로 넘겨주는 실행 가능한 코드를 콜백이라한다.
- 콜백
    - 다른 코드의 인수로서 넘겨주는 실행가능한 코드
- 자바에서 콜백
    - 람다
    - 익명 내부 클래스(하나의 인터페이스를 구현해 하나의 메서드만 가진)


### 템플릿 콜백 패턴
- Context가 템플릿 역할을 하고 Strategy 부분이 콜백으로 넘어온다.
- 전략패턴에서 템플릿과 콜백이 강조된 패턴
- JdbcTemplate, RestTemplate, ...
    - `XXXTeplate`가 있다면 콜백 템플릿 패턴이다.

```java

public interface Callback {
    void call();
}

@Slf4j
public class TimeLogTemplate {

    public void execute(Callback callback) {
        long startTime = System.currentTimeMillis();

        callback.call();
        long endTime = System.currentTimeMillis();
        log.info("resultTime {}", endTime - startTime);
    }
}

@Slf4j
public class TemplateCallbackTest {

    @Test
    void callbackV1() {
        TimeLogTemplate template = new TimeLogTemplate();
        template.execute(() -> log.info("비지니스로직 1"));
        template.execute(() -> log.info("비지니스로직 2"));
    }
}

```

### 한계
- 코드를 다 분리해도, 원본코드 들을 다 수정해야한다.
- 클래스가 수백개가 있다면 그 수백개는 결국 건드려야한다.
- Proxy 공부해보자.