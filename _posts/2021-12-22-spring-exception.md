---
layout: post
title:  "client-exception-spring"
date:   2021-12-22 22:30:21 +0900
categories: java, spring, exception
---

> Exception

## 예외
- 자바 직접 실행
    - 자바의 메인 메서드를 직접 실행하는 경우 메인이라는 스레드가 실행된다. 실행 도중에 예외를 잡지 못하고 처음 실행한 메인 메서드를 넘어서 예외가 던져지면 예외정보를 남기고, 스레드는 종료된다.
- 웹 어플리케이션
    - 요청별로 별도의 스레드가 할당되고, 서블릿 컨테이너안에서 실행된다.
    - 예외가 발생했을 때, try catch가 실행되면 문제가 없지만, 예외를 잡지못하고, 서블릿 밖으로 전달되면 `컨트롤러 (예외발생) -> 인터셉터 -> 서블릿 -> 필터 -> WAS`
    - response.sendError(HTTP 상태 코드, 오류 메시지)
        - 오류가 발생했을 때 `HttpServletResponse`가 제공하는 메서드 호출, 서블릿 컨테이너에게 오류가 발생했다고 알려줌
    - response 내부적으로 오류 상태를 가지고 있다가 응답 전에 호출되었는지 확인하고, 설정된 에러페이지를 보내준다.

- 설정된 에러페이지 처리하기

```java
public class WebServerCustomerizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {

        // 두번째 파라미터는 컨트롤러를 가르킨다.
        final ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/400");
        final ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");
        // Runtime 자식 에러들도 다 이곳으로 처리된다.
        final ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");
        
        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}

```

- 서블릿 예외처리 (오류페이지 작동원리)
    1. WAS는 예외처리하는 오류페이지 정보를 확인
    2. new ErrorPage(RuntimeException.class, "/error-page/500");
    3. RuntimeException 예외가 WAS까지 전달되면, WAS는 오류페이 정보를 확인한다. 확인해보니 `/error-page/500`이 확인되어 오류페이지를 다시 /error-page/500 요청한다.
- 정리
    - 예외가 발생하면 WAS까지 전파
    - WAS는 오류 페이지 경로를 찾아서 내부 오류페이지를 호출한다. 오류페이지 경로로 필터 서블릿 인터셉터 컨트롤러가 `모두 다시 호출된다.`
    
- 서블릿 예외처리(필터)
    - 오류가 발생하면 오류페이지를 출력하기 위해 was가 내부에서 다시 한번 호출이된다. (다시 호출 된다는 말은 filter->servlet->interceptor를 다시 탄다) 다시 호출되는것은 매우 비효율적이다. 그래서 클라이언트로부터 발생한 정상요청인지, 오류 페이지를 출 력하기 위한 내부 요청인지 구분해야해서 Servlet은 `DispatcherType`을 제공한다.
    - `DispatcherType`
        - 고객이 처음 요청하면 dispatcherType=REQUEST(default)
        - 에러는 dispatcherType=ERROR
        - 그 외 FORWARD, INCLUDE, ASYNC가 있다.

- 필터 & 인터셉터

```java

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "*.ico", "/error", "/error-page/**"); // error-page가 들어오면 인터셉터 타지 않는다.
    }

//    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR); // dispatcheType.ERROR가 들어오면 필터를 타기때문에 제거하면 타지 않는다.
        return filterRegistrationBean;
    }


```

### 스프링부트
- ErrorPage 객체를 자동으로 등록, `/error`라는 경로로 기본 오류페이지를 설정
- new ErrorPage("/error"), 상태 코드를 지정하지 않으면 기본 오류페이질 설정된다.
- `BasicErrorController`라는 스프링 컨트롤러를 자동으로 등록한다.
    - ErrorPage에서 등록한 /error를 매핑해서 처리하는 컨트롤러
    - 우선순위
        - 뷰템플릿(template)
        - 정적리소스(static, public)
        - 적용 대상이 없을 때 error.html
    - BasicController가 제공(보안에 좋지 않다.)

```html

    <ul>
        <li>오류 정보</li>
        <ul>
            <li th:text="|timestamp: ${timestamp}|"></li>
            <li th:text="|path: ${path}|"></li>
            <li th:text="|status: ${status}|"></li>
            <li th:text="|message: ${message}|"></li>
            <li th:text="|error: ${error}|"></li>
            <li th:text="|exception: ${exception}|"></li>
            <li th:text="|errors: ${errors}|"></li>
            <li th:text="|trace: ${trace}|"></li>
        </ul>
        </li>
    </ul>

```

- ErrorMvcAutoConfiguration이라는 클래스가 오류페이지를 자동으로 등록하는 역할을 한다.

```java

@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

```

