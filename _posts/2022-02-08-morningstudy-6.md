---
layout: post
title:  "@Aspect Aop"
date:   2022-02-08 07:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 11번 째

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