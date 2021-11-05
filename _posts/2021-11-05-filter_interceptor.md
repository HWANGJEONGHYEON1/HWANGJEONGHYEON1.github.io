---
layout: post
title: "필터와 인터셉터"
date:  2021-11-05 20:20
categories: spring, filter, interceptor
tags: [spring, filter, interceptor]
---

## 필터, 인터셉터

### 공통
- 어플리케이션 여러 로직에서 공통으로 관심이 있는 것을 관심사. 등록, 수정, 삭제 조회 등의 여러 로직으로 인증에 대해서 관심을 가지고 있다.
- 공통 관심사는 AOP를 통해 해결할 수 있지만, 웹과 관련된 관심사는 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋음.
- 웹과 관련된 공통의 관심사는 HTTP Header, Url 정보들이 필요한대, 서블릿 필터나 인터셉터는 HttpServletRequest를 제공

### 필터
- 요청 흐름
    > Http 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
- 특정 URL Pattern에 적용할 수 있다.
    > /api/login 등
- 스프링에서 서블릿 필터는 `디스펫처 서블릿`
- 필터 인터페이스
```java
    public interface Filter {

    /**
     * Called by the web container to indicate to a filter that it is being
     * placed into service. The servlet container calls the init method exactly
     * once after instantiating the filter. The init method must complete
     * successfully before the filter is asked to do any filtering work.
     * <p>
     * The web container cannot place the filter into service if the init method
     * either:
     * <ul>
     * <li>Throws a ServletException</li>
     * <li>Does not return within a time period defined by the web
     *     container</li>
     * </ul>
     * The default implementation is a NO-OP.
     *
     * @param filterConfig The configuration information associated with the
     *                     filter instance being initialised
     *
     * @throws ServletException if the initialisation fails
     */
    // 필터 초기화 메서드
    public default void init(FilterConfig filterConfig) throws ServletException {} 

    /**
     * The <code>doFilter</code> method of the Filter is called by the container
     * each time a request/response pair is passed through the chain due to a
     * client request for a resource at the end of the chain. The FilterChain
     * passed in to this method allows the Filter to pass on the request and
     * response to the next entity in the chain.
     * <p>
     * A typical implementation of this method would follow the following
     * pattern:- <br>
     * 1. Examine the request<br>
     * 2. Optionally wrap the request object with a custom implementation to
     * filter content or headers for input filtering <br>
     * 3. Optionally wrap the response object with a custom implementation to
     * filter content or headers for output filtering <br>
     * 4. a) <strong>Either</strong> invoke the next entity in the chain using
     * the FilterChain object (<code>chain.doFilter()</code>), <br>
     * 4. b) <strong>or</strong> not pass on the request/response pair to the
     * next entity in the filter chain to block the request processing<br>
     * 5. Directly set headers on the response after invocation of the next
     * entity in the filter chain.
     *
     * @param request  The request to process
     * @param response The response associated with the request
     * @param chain    Provides access to the next filter in the chain for this
     *                 filter to pass the request and response to for further
     *                 processing
     *
     * @throws IOException if an I/O error occurs during this filter's
     *                     processing of the request
     * @throws ServletException if the processing fails for any other reason
     */
     // 고객의 요청이 올때마다 해당 메서드 호출
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    /**
     * Called by the web container to indicate to a filter that it is being
     * taken out of service. This method is only called once all threads within
     * the filter's doFilter method have exited or after a timeout period has
     * passed. After the web container calls this method, it will not call the
     * doFilter method again on this instance of the filter. <br>
     * <br>
     *
     * This method gives the filter an opportunity to clean up any resources
     * that are being held (for example, memory, file handles, threads) and make
     * sure that any persistent state is synchronized with the filter's current
     * state in memory.
     *
     * The default implementation is a NO-OP.
     */
    public default void destroy() {}
}
```

- 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 관리
- 소스코드 확인 
```java
@Slf4j
public class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("log filter doFilter");

        final HttpServletRequest request1 = (HttpServletRequest) request;
        final String requestURL = request1.getRequestURI();
        String uuid = UUID.randomUUID().toString();

        try {
            log.info("REQUEST [{}][{}]", uuid, requestURL.toString());
            chain.doFilter(request, response);
        } catch (Exception e) {
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}]", uuid, requestURL);
        }

    }

    @Override
    public void destroy() {
        log.info("filter destroy");
    }
}

@Configuration
public class WebConfig {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistration = new FilterRegistrationBean<>();;
        filterRegistration.setFilter(new LogFilter());
        filterRegistration.setOrder(1);
        filterRegistration.addUrlPatterns("/*");

        return filterRegistration;
    }
}
```