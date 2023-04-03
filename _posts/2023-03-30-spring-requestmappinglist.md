---
layout: post
title:  "스프링 requestmapping 값 가져오기"
date:   2023-01-31 23:20:21 +0900
categories: kafka, spring
---

> 사용하지 않는 API를 제거하기 위해 API 목록 뽑기

<h3>Code</h3>
```java

@Component
@Slf4j
@RequiredArgsConstructor
public class ApiCheckTasks implements ApplicationListener<ContextRefreshedEvent> {

    private final RequestMappingHandlerMapping requestMappingHandlerMapping;

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        Map<RequestMappingInfo, HandlerMethod> handlerMethods = requestMappingHandlerMapping.getHandlerMethods();

        List<String> apiList = new ArrayList<>();

        for (RequestMappingInfo handlerMethod : handlerMethods.keySet()) {
            apiList.addAll(handlerMethod.getPatternsCondition().getPatterns());
        }
    }
}

@Aspect
@Component
@Slf4j
@RequiredArgsConstructor
public class ApiUsageMonitor {
    private final ApiList apiList;

    @Before("execution(* com.wemakeprice.board.api.controller.*.*(..))")
    public void logBefore(JoinPoint joinPoint) {
        // HttpServletRequest 객체 가져오기
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
        // 요청 URI 가져오기
        String requestUri = request.getRequestURI();
        // 가져온 요청 URI를 로그로 출력
        if (apiList.remove(requestUri)) {
            log.info("removed api: {}", requestUri);
        }
    }
}

@Component
@Getter
@ToString
public class ApiList {

    private final Set<String> apiList = new HashSet<>();

    public int size() {
        return apiList.size();
    }

    public boolean remove(String requestUri) {
        for (Iterator<String> it = apiList.iterator(); it.hasNext(); ) {
            String mappingUrl = it.next();
            if (new AntPathMatcher().match(mappingUrl, requestUri)) {
                it.remove(); // 일치하는 URL을 리스트에서 제거합니다.
                return true;
            }
        }

        return false;
    }
}


```

<h3>설명</h3>
- ApplicationListener를 쓴 이유?
  - 모든 빈 주입이 완료된 후에 초기화 작업을 수행할 수 있는 방법입니다.
  - Application 인터페이스를 구현한 클래스에서, ContextRefreshedEvent 이벤트를 처리하면, 스프링 컨텍스트가 초기화가 되고 나서 모든 빈이 생성되고 주입되었을 때, 이벤트 발생가 발생합니다.
- 마치 `@PostConstruct` 어노테이션처럼 작동하는대, 빈 주입이 디 이루어지고 나서 어플리케이션 실행시점에 값을 API 리스트를 가져오는 것 처럼 작동되게 됩니다.
- AspectJ로 들어온 요청에 맞으면 List에서 제거합니다.
- `AntPathMatcher`를 쓰는 이유는 Pathvariable로 들어오는 값이 있을 수 있기 때문에 사용합니다. 
  - 예) /api/{number}