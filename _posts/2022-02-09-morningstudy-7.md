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

### 적용
- 코드

```java
// annotation 생성
// 간헐적으로 서버에서 에러가 났을 시 재시도(value를 꼭 지정해야함 안그럼 self 디도스를 할 수 있다.)
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Retry {
    int value() default 3;
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Trace {
}

@Slf4j
@Aspect
public class RetryAspect {

    @Around("@annotation(retry)")
    public Object doRetry(ProceedingJoinPoint joinPoint, Retry retry) throws Throwable {
        log.info("[retry] {}, args = {}", joinPoint.getSignature(), retry);
        int maxRetry = retry.value();
        Exception exceptionHolder = null;

        for (int i = 0; i <= maxRetry; i++) {
            try {
                log.info("[retry] try count {}/{}", i, maxRetry);
                return joinPoint.proceed();
            } catch (Exception e) {
                exceptionHolder = e;
            }
        }
        throw exceptionHolder;
    }
}

@Aspect
@Slf4j
public class TraceAspect {

    @Before("@annotation(hello.aop.exam.annotation.Trace)")
    public void doTrace(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        log.info("[trace] {}, args={}", joinPoint.getSignature(), args);
    }
}


@Repository
public class ExamRepository {

    private static int seq = 0;

    @Trace
    @Retry(4)
    public String save(String itemId) {
        seq++;
        if (seq % 5 == 0) {
            throw new IllegalStateException("예외발생");
        }
        return "ok";
    }
}

@Service
@RequiredArgsConstructor
public class ExamService {

    private final ExamRepository examRepository;

    @Trace
    public void request(String itemId) {
        examRepository.save(itemId);
    }
}

@SpringBootTest
@Slf4j
@Import({TraceAspect.class, RetryAspect.class})
public class ExamTest {

    @Autowired
    ExamService examService;

    @Test
    void test() {
        for (int i = 0; i < 5; i++) {
            log.info("client request i = {}", i);
            examService.request("data" + i);
        }
    }
}

```

### 프록시 내부 호출 문제
- AOP를 적용하면 스프링 대상 객체 대신에 프록시가 스프링 빈으로 등록한다. 따라서 스프링은 의존관계 주입 시에 항상 프록시 객체를 주입한다.
- 프록시 객체가 주입되기 때문에 대상 객체를 직접 호출하는 문제는 일반적으로 발생하지 않는다.
- 대상 객체 내분에서 메서드호출이 발생하면 프록시를 거치지 않고 대상 객체를 직접 호출하는 문제가 발생
- 코드
    ```java
    @Test
    void external() {
        callServiceV0.external();
    }

    @Before("execution(* hello.aop.internalcall..*.*(..))")
    public void doLog(JoinPoint joinPoint) {
        log.info("aop {}", joinPoint.getSignature());
    }

    @Component
    @Slf4j
    public class CallServiceV0 {

        public void external() {
            log.info("call external");
            this.internal(); // 내부에서 호출
        }

        public void internal() {
            log.info("call internal");
        }
    }
    ```

### 대안1
- 자기 자신을 의존관계주입 받는 것
- 코드

    ```java

    private CallServiceV1 callServiceV1;

    @Autowired
    public void setCallServiceV1(CallServiceV1 callServiceV1) {
        log.info("SETTER {}", callServiceV1.getClass());
        this.callServiceV1 = callServiceV1;
    }

    ```

### 대안2
- 지연조회
- applicationContext 활용

    ```java
    //private final ApplicationContext applicationContext; // 너무거대함
    private final ObjectProvider<CallServiceV2> objectProvider;


    public void external() {
        log.info("call external");
        //CallServiceV2 bean = applicationContext.getBean(CallServiceV2.class);
        CallServiceV2 callServiceV2 = objectProvider.getObject();
        callServiceV2.internal();
    }
    ```

### 대안3
- 구조변경
    - 내부호출이 발생하지 않도록 한다.
    - internal 메소드를 분리
- 코드

    ```java
    @Component
    @Slf4j
    @RequiredArgsConstructor
    public class CallServiceV3 {

        private final InternalService internalService;

        public void external() {
            log.info("call external");
            internalService.internal();
        }
    }

    @Component
    @Slf4j
    public class InternalService {

        public void internal() {
            log.info("call internal");
        }
    }


    ```

### 프록시 기술, 한계

- JDK 동적 프록시와 CGLIB를 사용해서 AOP 프록시를 만드는 방법에는 장단점이 있음.
- JDK 동적프록시는 인터페이스가 필수, 인터페이스를 기반으로 프록시 생성
- CGLIB는 구체클래스를 통해 프록시를 생성
- JDK 동적 프록시 `한계`
- 코드 
    ```java

    @Test
    void jdkProxy() {
        MemberServiceImpl target = new MemberServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.setProxyTargetClass(false); // JDK 동적 프록시

        // 프록시 캐스팅 성공
        MemberService memberServiceProxy = (MemberService) proxyFactory.getProxy();

        //JDK 동적 프록시를 구현 클래스로 캐시팅 시도 실패, public class ClassCastException extends RuntimeException {


        Assertions.assertThrows(ClassCastException.class, () -> {
            MemberServiceImpl memberServiceImpl = (MemberServiceImpl) memberServiceProxy;
        });

    }

    @Test
    void cgLibProxy() {
        MemberServiceImpl target = new MemberServiceImpl();
        ProxyFactory proxyFactory = new ProxyFactory(target);
        proxyFactory.setProxyTargetClass(true); // JDK 동적 프록시

        // 프록시 캐스팅 성공
        MemberService memberServiceProxy = (MemberService) proxyFactory.getProxy();

        log.info("target = {}", memberServiceProxy.getClass());
        //CGLIB 프록시를 구현 클래스로 캐스팅 성공
        MemberServiceImpl memberServiceImpl = (MemberServiceImpl) memberServiceProxy;
    }
    ```

- 의존관계 주입
    - JDK는 인터페이스에만 의존하게 되는대, 구체클래스를 의존관계에 할 때 에러가 발생한다.
    - CGLIB만 사용하면 에러안나는거 아닌가?
        - 문제점
            - 대상 클래스에 기본 생성자 필수
            - 생성자 2번 호출 문제
            - final 키워드클래스 사용 불가
- 스프링 권장
    - 스프링부트 2.0 부터는 CGLIB가 기본으로 사용
    - `proxyTargetClass=true`로 설정해서 사용
    - 인터페이스가 있어도 항상 CGLIB를 사용

