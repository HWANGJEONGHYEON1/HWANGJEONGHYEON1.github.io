---
layout: post
title:  "포인트컷 지시자"
date:   2022-02-10 07:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 14번 째

# 포인트 컷 지시자

### 종류
- execution : 메서드 실행 조인포인트를 매칭. 스프링 AOP에서 가장 많이 사용
- within : 특정 타입 내의 조인포인트를 매칭
- args : 인자가 주어진 타입의 인스턴스인 조인 포인트
- this : 스프링 빈 객체(스프링 AOP 프록시)를 대상으로 하는 조인 포인트
- target : Target 객체(스프링 AOP 프록시가 가르키는 실제 대상)를 대상으로 하는 조인포인트
- @Target : 실행 겍체의 클래스에 주어진 타입의 애노테이션이 있는 조인 포인트
- @within : 주어진 애노테이션이 있는 타입 내 조인포인트
- @annotation : 메서드가 주어진 애노테테이션을 가지고 있는 조인포인트를 매칭


### execution
- execution( 접근제어자 ? 반환타입 선언타입?메서드이름(파라미터) 예외?)
- 메서드 실행 조인포인트를 매칭
- ?는 생략가능
- `*` 같은 패턴을 지정할 수 있다.
- 매칭조건
    - 접근제어자?: `public`
    - 반환타입 : String
    - 선언타입?: hello.aop.member.MemberServiceImpl.hello(String)
    - 메서드이름: hello
    - 파라미터: (String)
    - 예외?: 생략
    - 코드

    ```java
    @Test
    void exactMatch() {
        // public java.lang.String hello.aop.member.MemberServiceImpl.hello(java.lang.String)
        pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");
        Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }

    @Test
    void allMatch() {
        pointcut.setExpression("execution(* *(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class));
    }

    @Test
    void nameMatch() {
        pointcut.setExpression("execution(* hello(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class));
    }

    @Test
    void nameMatchStar() {
        pointcut.setExpression("execution(* *ll*(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class));
    }

    @Test
    void packageExactSubMatchFalse() {
        pointcut.setExpression("execution(* hello.aop.member.*.*(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }

    @Test
    void superTypeMatch() {
        pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }

    @Test
    void typeMatchInternal() throws NoSuchMethodException {
        pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");

        Method internal = MemberServiceImpl.class.getMethod("internal", String.class); // 부모의 메서드를 가지지 않은 자식만의 메서드
        assertThat(pointcut.matches(internal, MemberServiceImpl.class)).isFalse();
    }

    @Test
    void argsNoMatch() {
        // String type의 파라미터 허용
        pointcut.setExpression("execution(* *())");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
    }

    // 정확히 하나의 파라미터 허용, 모든타입
    @Test
    void argsMatchStar() {
        pointcut.setExpression("execution(* *(*))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }

    // 정확히 하나의 파라미터 허용, 몇개의 타입이든 노상관
    @Test
    void argsMatchStarAll() {
        pointcut.setExpression("execution(* *(..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }

    @Test
    void argsMatchComplex() {
        // (String), (String, xxx), (String,xxx, xxx)...
        pointcut.setExpression("execution(* *(String, ..))");
        assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }    
    ```

### within & args
- within : execution에서 타입부분만 사용
- args : 부모타입을 허용하지만 , execution은 정확히 타입매칭이 되어야한다.

### @target, @within
- @target: 실행 객체의 클래스에 주어진 타입의 애노테이션이 있는 조인포인트
- @within: 주어진 애노테이션이 있는 조인포인트
- 차이 
    - @target은 인스턴스의 모든 조인 포인트를 적용(부모클래스까지)
    - @within은 해당 타입 내에 있는 메서드만 조인 포인트 적용

### @annotation, @args
- @annotation: 메서드가 주어진 에노테이션을 가지고 있는 조인포인트를 매칭


### 매개변수 전달
- 포인트 컷의 이름과 매게변수의 이름을 맞추어야함.
- 추가로 타입이 메서드에 지정한 타입으로 제한된다.
- 코드

```java
 @Slf4j
    @Aspect
    static class ParametetAspect {
        @Pointcut("execution(* hello.aop.member..*(..))")
        private void allMember() {}

        @Around("allMember()")
        public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
            Object arg1 = joinPoint.getArgs()[0];
            log.info("[logArgs1]{}, args = {}", joinPoint.getSignature(), arg1);
            return joinPoint.proceed();
        }

        @Around("allMember() && args(arg, ..)")
        public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
            log.info("[logArgs1]{}, args = {}", joinPoint.getSignature(), arg);
            return joinPoint.proceed();
        }

        @Before("allMember() && args(arg, ..)")
        public void logArgs3(String arg) {
            log.info("logargs3 => args = {}", arg);
        }

        @Before("allMember() && this(obj)")
        public void thisArgs(JoinPoint joinPoint, MemberService obj) {
            log.info("[this]{}, obj={}", joinPoint.getSignature(), obj.getClass());
        }

        @Before("allMember() && target(obj)")
        public void targetArgs(JoinPoint joinPoint, MemberService obj) {
            log.info("[target]{}, obj={}", joinPoint.getSignature(), obj.getClass());
        }

        @Before("allMember() && @target(annotation)")
        public void targetArgs1(JoinPoint joinPoint, ClassAop annotation) {
            log.info("[target]{}, obj={}", joinPoint.getSignature(), annotation);
        }
    }
```
