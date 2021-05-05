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

* 토큰 생성 및 검증 

```java

@RequiredArgsConstructor
@Component
public class JwtTokenProvider {

    private String secretKey = "hanshinKey";

    //토큰 유효 시간 30분
    private long tokenValidTime = 30 * 60 * 1000L;

    private final UserDetailsService userDetailsService;

    @PostConstruct
    protected void init() {
        secretKey = Base64.getEncoder()
                .encodeToString(secretKey.getBytes());
    }

    public String createToken(String userPk, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(userPk); // JWT payload에 저장
        claims.put("roles", roles);
        Date now = new Date();

        return Jwts.builder()
                .setClaims(claims) // 정보저장
                .setIssuedAt(now) // 토큰 발행시간 정보
                .setExpiration(new Date(now.getTime() + tokenValidTime)) // 만료시간
                .signWith(SignatureAlgorithm.HS256, secretKey) // 사용할 암호화 알고리즘
                .compact();
    }

    // JWT 토큰에서 인증 정보 조회
    public Authentication getAuthentication(String token) {
        UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUserPk(token));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    // 토큰에서 회원 정보 추출
    private String getUserPk(String token) {
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }

    // Request의 헤더에서 token 값을 가져온다. "X-AUTH-TOKEN" : "TOKEN값'
    public String resolveToken(HttpServletRequest request) {
        return request.getHeader("X-AUTH-TOKEN");
    }

    // 토큰의 유효성 및 만료일자 확인
    public boolean validateToken(String jwtToken) {
        try {
            Jws<Claims> claimsJws = Jwts.parser()
                                        .setSigningKey(secretKey)
                                        .parseClaimsJws(jwtToken);
            
            return !claimsJws.getBody()
                    .getExpiration()
                    .before(new Date());
        } catch (Exception e) {
            return false;
        }
    }
}
```

### Ref.
* <https://sjh836.tistory.com/165>
* <https://sieunlim.tistory.com/19>
* <https://mangkyu.tistory.com/57>
