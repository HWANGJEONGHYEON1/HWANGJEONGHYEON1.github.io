---
layout: post
title:  "@Aspect Aop"
date:   2022-02-08 07:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 12번 째

# @Aspect Aop
> 스프링 어플리케이션에 프록시를 적용하려면 Advisor를 만들어서 스프링 빈으로 등록하면 된다. 그러면 자동 프록시 생성기가 모두 자동으로 처리해준다.
- `@Apsect` 어노티에션은 매우 편리하게 포인트컷과 어드바이스로 구성되어 있는 어드바이저 생성 기능을 지원한다.
- 코드

```java
@Slf4j
@Aspect
@RequiredArgsConstructor
public class LogTraceAspect {

    private final LogTrace logTrace;

    @Around("execution(* hello.proxy.app..*(..))") // 포인트 컷
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        // 어드바이스 로직
        TraceStatus status = null;
        try {
            String message = joinPoint.getSignature().toShortString();
            status = logTrace.begin(message);

            Object result = joinPoint.proceed(); // 실제 호출대상을 호출 (target)

            logTrace.end(status);
            return result;
        } catch (Exception e) {
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```


### AOP

- `관점지향 프로그래밍`
- 핵심기능과 부가기능
- 스프링이 제공하는 어드바이저, 어드바이스, 포인트 컷을 가지고 있어 하나의 개념상의 aspect다
- `OOP를 대체하기 위한 것이 아니라, 횡단 관심사를 깔끔하게 처리하기 어려운 OOP의 부족한 부분을 보조하는 목적으로 개발됨`
- 횡단 관심사
    - 오류 검사 및 처리
    - 동기화
    - 성능 최적화(캐싱)
    - 모니터링 및 로깅

### AOP 적용방식

- 3가지
    - 컴파일 시점
        - 원본 로직에 부가 기능 로직이 필요 : weaving 
        - 특별한 컴파일러가 필요
    - 클래스 로딩 시점
        - .class 파일을 jvm내부의 클래스 로더에 보관. 중간에서 .class 파일을 조작한 다음 jvm에 올린다: 자바 instrument(모니터링 툴들이 많이 사용)
    - 런타임 시점
        - 컴파일이 다끝나고, 클래스로더에 클래스도 다 올라가서 이미 자바가 실행되고 난 다음을 말한다. main 메서드가 이미 실행된 다음 : 프록시 방식의 AOP
        - 스프링만 있으면 실행가능
- 적용위치
    - 적용 가능 지점(조인 포인트): 생성자, 필드 값 접근, static 메서드, 메서드 실행(aspectJ 라이브러리만 가능)
    - 프록시 방식을 사용하는 스프링 AOP는 `메서드 실행지점`에만 AOP를 적용할 수 있다.
        - 스프링 AOP 컨테이너가 관리할 수 있는 `스프링 빈에만` 적용 가능하다.

### AOP 용어

- 조인포인트
    - 어드바이스가 적용할 수 있는 위치
    - 추상적인 개념, AOP를 적용할 수 있는 모든 지점
    - 조인포인트는 항상 METHOD 실행 지점으로 제한
- 포인트 컷
    - 조인 포인트 중 어드바이스가 적용될 위치를 선별하는 기능
    - AspectJ 표현식을 사용해서 지정
    - 프록시를 사용하는 스프링 AOP는 메서드 실행지점만 선별 가능
- 타겟
    - 어드바이스를 받는 객체, 포인트컷으로 결정
- 어드바이스
    - 부가기능
    - 특정 조인 포인트에서 Aspect에 의해 취해지는 조치
    - Around, Before, After후와 같은 다양한 종류의 어드바이스가 있음
- 애스팩트
    - 어드바이스 + 포인트컷 모듈화
    - @Aspect
    - 여러 어드바이스와 포인트컷이 함께 존재
- 어드바이저
    - 하나의 어드바이스와 하나의 포인트컷 으로 구성
- 위빙
    - 포인트 컷으로 결정한 타겟의 조인 포인트에 어드바이스를 적용하는 것
    - 위빙을 통해 핵심기능 코드에 영향을 주지 않고 부가 기능을 추가할 수 있음


### @Pointcut
- 메서드 이름과 파라미터를 합쳐서 포인트컷 시그니쳐라고 한다.
- 메서드의 반환 타입은 void
- 코드 내용은 비워둔다.
- @Around 어드바이스에서는 포인트컷을 지정해도 되지만, 포인트컷 시그니처를 사용해도 된다.
- private, public 같은 접근제어자는 내부에서만 사용하면 private을 사용해도 되지만, 다른 apect에서 참고하려면 public 이라고한다.


```java
    @Pointcut("execution(* hello.aop.order..*(..))") // pointCut
    private void allOrder(){} // pointCut signature

    @Around("allOrder()") // pointCut
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { // advice
        log.info("[log] {}", joinPoint.getSignature()); // joinpoint sigature
        return joinPoint.proceed();
    }

```

- 트랜잭션 예제

```java

    @Pointcut("execution(* hello.aop.order..*(..))") // pointCut
    private void allOrder(){} // pointCut signature

    // class 이름 패턴이 *Service
    @Pointcut("execution(* *..*Service.*(..))")
    private void allService() {}

    @Around("allOrder()") // pointCut
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { // advice
        log.info("[log] {}", joinPoint.getSignature()); // joinpoint sigature
        return joinPoint.proceed();
    }

    //hello.aop.order 패키지와 하위 패키지 이면서 클래스이름 패턴이 *Service인 것
    @Around("allOrder() && allService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object proceed = joinPoint.proceed();
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return proceed;
        } catch (Exception e) {
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }
```

### 어드바이스 순서
- @Aspect 단위로 순서가 됨
- @Order로 순서 가능 클래스 단위로 적용가능.
- 소스

```java

@Slf4j
public class AspectV5Order {
    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("hello.aop.order.aop.Pointcuts.allOrder()") // pointCut
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable { // advice
            log.info("[log] {}", joinPoint.getSignature()); // joinpoint sigature
            return joinPoint.proceed();
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect {
        @Around("hello.aop.order.aop.Pointcuts.orderAndService()")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
            try {
                log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
                Object proceed = joinPoint.proceed();
                log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
                return proceed;
            } catch (Exception e) {
                log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
                throw e;
            } finally {
                log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
            }
        }
    }
}
```

### 어드바이스 종류

```java

@Around("hello.aop.order.aop.Pointcuts.orderAndService()")
    public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
        try {
            // @Before
            log.info("[트랜잭션 시작] {}", joinPoint.getSignature());
            Object proceed = joinPoint.proceed();
            //@AfterReturning
            log.info("[트랜잭션 커밋] {}", joinPoint.getSignature());
            return proceed;
        } catch (Exception e) {
            // @AfterThrowing
            log.info("[트랜잭션 롤백] {}", joinPoint.getSignature());
            throw e;
        } finally {
            // @After
            log.info("[리소스 릴리즈] {}", joinPoint.getSignature());
        }
    }

    @Before("hello.aop.order.aop.Pointcuts.orderAndService()")
    public void before(JoinPoint joinPoint) {
        log.info("[before] {}", joinPoint.getSignature());
    }

    @AfterReturning(value = "hello.aop.order.aop.Pointcuts.orderAndService()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        log.info("[afterReturning] {}, result = {}", joinPoint.getSignature(), result);
    }

    @AfterThrowing(value = "hello.aop.order.aop.Pointcuts.orderAndService()", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
        log.info("[AfterThrowing] {}, ex = {}", joinPoint.getSignature(), ex);
    }

    @After(value = "hello.aop.order.aop.Pointcuts.orderAndService()")
    public void after(JoinPoint joinPoint) {
        log.info("[After] {}", joinPoint.getSignature());
    }

```