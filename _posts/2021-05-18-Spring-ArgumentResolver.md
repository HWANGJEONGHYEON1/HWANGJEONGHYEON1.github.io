---
layout: post
title:  "ArgumentResolver"
date:   2021-05-18 20:15
categories: spring, resolver
tags: [spring, resolver]
---

### ArgumentResolver

#### `ArgumentResolver` 란
- 컨트롤러에 들어오는 파라미터를 가공하거나, 파라미터를 추가해서 사용해야하는 경우에 사용

- 스프링에서 제공하는 ArgumentResolver를 사용하면 Controller에 파라미터에 대한 공통기능을 제공할 수 있다.

- 컨트롤러로 가기 전에 view에서 넘어온 데이터(정확히는 HttpRequest를 포함해서 다양한 데이터)를 가공해서, 컨트롤러로 전달하기 위함

- HandlerMethodArgumentResolver 사용


> 동작방식
- 클라이언트 Request 요청
- DispatcherServlet 요청 처리
- 요청에 대한 Handler Mapping
    - RequestMapping 매칭 -> RequestMappingHandlerAdapter
    - intercepter 처라
    - argumentResolver 처리
    - MessageConverter 처리
- controller 처리 

> 프로젝트에 구현한 코드
- 해당 클래스를 @LoginUser 어노테이션으로 생성

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {

}

```

- supportsParameter()
    - 메서드의 특정 파라미터를 지원하는지
- resolveArgument()
    - 메서드는 실제로 바인딩을 할 객체를 리턴

```java

@Slf4j
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final UserMapper mapper;

    public LoginUserArgumentResolver(UserMapper mapper) {
        this.mapper = mapper;
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = User.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return mapper.findByEmail(webRequest.getRemoteUser()).get();
    }
}

```

- resolver 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    public WebConfig(LoginUserArgumentResolver loginUserArgumentResolver) {
        this.loginUserArgumentResolver = loginUserArgumentResolver;
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(loginUserArgumentResolver);
    }
}
```

- 로그인 후 정보가져오기

```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class IndexController {

    @GetMapping(value = {"/", "/index"})
    public String index(Model model, @LoginUser User user) {
        final String username = SecurityUtil.getCurrentUsername().get();

        if (!username.equals("anonymousUser")) {
            log.info(user.getName());
            log.info(user.getEmail());
            model.addAttribute("userName", username);
        }

        return "index";
    }
}

```
