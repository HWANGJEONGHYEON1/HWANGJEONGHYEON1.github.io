---
layout: post
title:  "Spring-Security & JWT 구현 - 1"
date:   2021-05-02 01:15
categories: spring,security
tags: [security, jwt]
---
### Spring-Security 구현 - 1

* 프로젝트에 Spring Security, JWT 구현한다.

> WebConfig
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/static/", "classpath:/public/", "classpath:/",
            "classpath:/resources/", "classpath:/META-INF/resources/", "classpath:/META-INF/resources/webjars/" };

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // "/"에 해당하는 url-mapping을 index로 넘겨준다.
        registry.addViewController("/").setViewName("forward:/index");

        //우선순위 젤 높게
        registry.setOrder(Ordered.HIGHEST_PRECEDENCE);
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations(CLASSPATH_RESOURCE_LOCATIONS);
    }

}
```

> SecurityConfig
```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(WebSecurity webSecurity) {
        webSecurity.ignoring()
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                .authorizeRequests()
                // 토큰을 활용하는 경우 모든 요청에 대해 접근 가능하도록 함
                .anyRequest()
                .permitAll()
                .and()
                .sessionManagement()
                // 토큰을 활용하면서 세션이 필요없음  --> STATELESS로 설정하여 세션사용하지 않음
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .formLogin()
                .disable();
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

* 로그인 요청
* UserPasswordAuthenticationToken 발급

```java

public class CustomAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    public CustomAuthenticationFilter(AuthenticationManager authenticationManager) {
        super.setAuthenticationManager(authenticationManager);
    }

    @Override
    public Authentication attemptAuthentication(final HttpServletRequest request, final HttpServletResponse response) throws AuthenticationException {
        final UsernamePasswordAuthenticationToken authRequest;
        try {
            final User user = new ObjectMapper().readValue(request.getInputStream(), User.class);
            authRequest = new UsernamePasswordAuthenticationToken(user.getId(), user.getPwd());
        } catch (Exception e) {
            throw new InputNotFoundException();
        }

        setDetails(request, authRequest);
        return this.getAuthenticationManager().authenticate(authRequest);
    }

}
```

* 로그인 요청
* UserPasswordAuthenticationToken 발급
* Filter가 수행 된 후 처리될 handler를 bean으로 등록하고 필터에 추가
```java

@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class JwtTokenProvider {
    private static final long ACCESS_TOKEN_EXPIRE_TIME = 1000 * 60 * 30;

    private static final String secretKey = "hanshinKeyJwtToken";

    public static String generateJwtToken(User user) {
        JwtBuilder builder = Jwts.builder()
                .setSubject(user.getEmail())
                .setHeader(createHeader())
                .setClaims(createClaims(user))
                .setExpiration(createExpiration())
                .signWith(SignatureAlgorithm.HS256, createSignKey());

        return builder.compact();

    }

    private static Key createSignKey() {
        byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary(secretKey);
        return new SecretKeySpec(apiKeySecretBytes, SignatureAlgorithm.HS256.getJcaName());
    }

    private static Date createExpiration() {
        long now = (new Date()).getTime();
        Date accessTokenExpiresIn = new Date(now + ACCESS_TOKEN_EXPIRE_TIME);
        return accessTokenExpiresIn;

    }

    private static Map<String, Object> createClaims(User user) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("email", user.getEmail());
        claims.put("role", user.getUserRole());

        return claims;
    }

    private static Map<String, Object> createHeader() {
        Map<String, Object> header = new HashMap<>();
        header.put("type", "JWT");
        header.put("alg", "HS256");
        header.put("regDate", System.currentTimeMillis());

        return header;
    }
}
```

* 로그인 요청
* UserPasswordAuthenticationToken 발급
* Filter가 수행 된 후 처리될 handler를 bean으로 등록하고 필터에 추가
* loginSuccessHandler는 AuthenticationProvider를 통해 인증이 성공 될 경우 처리
* 로그인 성공 시 TokenProvider를 통해 토큰생성 후 response에 추가
* UsernamePasswordToken 발급
* UsernamePasswordToken을 Authentication Manager에게 전달
* 매니저는 실제로 인증을 처리할 여러 개의 AuthenticationProvider를 가지고 있음
* UsernamePasswordToken을 AuthenticationProvider에게 전달하여 인증과정 수행
* 스프링 시큐리티에서는 Username으로 DB에서 데이터를 조회한 다음, 비밀번호의 일치 여부를 검사하는 방식으로 작동
* 먼저 UsernamePasswordToken으로 아이디를 조회한다.

```java
@RequiredArgsConstructor
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserMapper mapper;

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        return mapper.findByEmail(email)
                .map(u -> new CustomUserDetails(u, Collections.singleton(new SimpleGrantedAuthority(u.getUserRole().getRole()))))
                .orElseThrow(() -> new UserNotFoundException(email));
    }
}
```


* 로그인 요청
* UserPasswordAuthenticationToken 발급
* Filter가 수행 된 후 처리될 handler를 bean으로 등록하고 필터에 추가
* loginSuccessHandler는 AuthenticationProvider를 통해 인증이 성공 될 경우 처리
* 로그인 성공 시 TokenProvider를 통해 토큰생성 후 response에 추가
* UsernamePasswordToken 발급
* UsernamePasswordToken을 Authentication Manager에게 전달
* 매니저는 실제로 인증을 처리할 여러 개의 AuthenticationProvider를 가지고 있음
* UsernamePasswordToken을 AuthenticationProvider에게 전달하여 인증과정 수행
* 스프링 시큐리티에서는 Username으로 DB에서 데이터를 조회한 다음, 비밀번호의 일치 여부를 검사하는 방식으로 작동
* 먼저 UsernamePasswordToken으로 아이디를 조회한다.
* 인증 처리된 후 인증된 토큰을 AuthenticationManager에게 전달
* CustomAuthenticationProvider에게 UserDetailsService를 통해 조회된 정보와 비밀번호가 일치하는 확인 후 일치하면 인증된 토큰 생성
* 사용자 비밀번호는 암호화되어 있어 PasswordEncorder로 암호화하여 DB 패스워드와 매칭
* CustomAuthenticationProvider 빈 등록


```java

@Configuration
@EnableWebSecurity
@AllArgsConstructor
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    public void configure(WebSecurity webSecurity) {
        webSecurity.ignoring()
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .formLogin()
                .disable()
                .addFilterBefore(customAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public CustomAuthenticationFilter customAuthenticationFilter() throws Exception {
        CustomAuthenticationFilter customAuthenticationFilter = new CustomAuthenticationFilter(authenticationManagerBean());
        customAuthenticationFilter.setFilterProcessesUrl("/login/login");
        customAuthenticationFilter.setAuthenticationSuccessHandler(customLoginSuccessHandler());
        customAuthenticationFilter.afterPropertiesSet();
        return customAuthenticationFilter;
    }

    @Bean
    public CustomLoginSuccessHandler customLoginSuccessHandler() {
        return new CustomLoginSuccessHandler();
    }

    @Bean
    public CustomAuthenticationProvider customAuthenticationProvider() {
        return new CustomAuthenticationProvider(bCryptPasswordEncoder());
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(customAuthenticationProvider());
    }
}
```



* 로그인 요청
* UserPasswordAuthenticationToken 발급
* Filter가 수행 된 후 처리될 handler를 bean으로 등록하고 필터에 추가
* loginSuccessHandler는 AuthenticationProvider를 통해 인증이 성공 될 경우 처리
* 로그인 성공 시 TokenProvider를 통해 토큰생성 후 response에 추가
* UsernamePasswordToken 발급
* UsernamePasswordToken을 Authentication Manager에게 전달
* 매니저는 실제로 인증을 처리할 여러 개의 AuthenticationProvider를 가지고 있음
* UsernamePasswordToken을 AuthenticationProvider에게 전달하여 인증과정 수행
* 스프링 시큐리티에서는 Username으로 DB에서 데이터를 조회한 다음, 비밀번호의 일치 여부를 검사하는 방식으로 작동
* 먼저 UsernamePasswordToken으로 아이디를 조회한다.
* 인증 처리된 후 인증된 토큰을 AuthenticationManager에게 전달
* CustomAuthenticationProvider에게 UserDetailsService를 통해 조회된 정보와 비밀번호가 일치하는 확인 후 일치하면 인증된 토큰 생성
* 사용자 비밀번호는 암호화되어 있어 PasswordEncorder로 암호화하여 DB 패스워드와 매칭
* CustomAuthenticationProvider 빈 등록
* 인증된 토큰을 AuthenticationFilter에 전달
* 인증이 성공하면 LoginSuccessHandler에서 Token 생성하여 반환함
* 인증된 토큰을 기반으로 JWT 반환
* 유효한 토큰 검증을 위한 인터셉터 추가




### Ref.
* <https://sjh836.tistory.com/165>
* <https://sieunlim.tistory.com/19>
* <https://mangkyu.tistory.com/57>
