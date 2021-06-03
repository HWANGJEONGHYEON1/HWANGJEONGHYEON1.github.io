---
layout: post
title:  "Spring-Security"
date:   2021-05-02 01:15
categories: spring,security
tags: [security, jwt]
---
### Spring-Security

### 프로젝트 시작 전 JWT & Spring Security에 대해 알아보고자 한다.

> 스프링 시큐리티

### 기본 동작방식
    - 서블릿의 여러 종류의 필터와 인터셉터를 이용해서 처리.
        - 필터와 인터셉트 차이
            - 필터는 스프링과 무관하게 서블릿 자원이고, 인터셉터는 스프링의 빈으로 관리되며 스프링의 컨텍스트 내에 속한다.
            - 인터셉터는 스프링의 내부에서 컨트롤러가 호출될 때 관여하기 때문에 스프링의 컨텍스트 내에 있는 모든 자원을 활용할 수 있다.

### 스프링 기반의 어플리케이션의 인증과 권한 부여를 사용하여 접근을 제어하는 프레임워크
    - 로그인 기능에서 인증을하고 관리자나 사용자를 구분하여 권한에 맞게 페이지 접속가능하다로 이해하면 될 것 같다.
    - `인증`, `권한`에 대한 부분을 Filter의 흐름에 따라 처리

### 스프링 시큐리티 쓰는 이유
    - 자체적 세션 체크
    - 모든 URL 가로채어 인증요구
    - CSRF 공격 방어
    - ServletApi 메소드 제공
    - 보안 관련 로직을 일일히 작성하지 않아도됨.

> 구조

![spring-security](/images/spring-security.png)

* Authentication
    - 현재 접근하는 주체의 정보와 권한을 담는 인터페이스
    - SecurityContext에 저장, SecurityContextHolder를 통해 Authentication에 접근

* SecurityContext
    - Authentication을 보관, SecurityContext를 통하여 Authentication 객체를 꺼낼 수 있음

* SecurityContextHolder
    - 보안주체의 세부 정보를 포함하여 응용 프로그램의 현재 보안 컨텍스트에 대한 세부정보가 저장

* UsernamePasswordAuthenticationToken
    - Authentication을 상송한 AbstractAuthenticationToken의 하위 클래스로 username이 `principal` 역할, password가 `Credential` 역할
    - 첫번째 생성자는 인증 전의 객체를 생성하고, 두번째는 인증이 완료된 객체를 생성

* `인증 처리 과정`
    1. username / password를 입력받아 UsernamePasswordAuthenticationToken 객체 생성
    2. 토큰은 검증을 받기 위해 AuthenticationManager에게 전달
    3. AuthenticationManager는 인증 성공 시 Authentication을 리턴
    4. Authentication은 SecurityContextHolder로 전달

```java
    public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
        // 주로 사용자의 username에 해당함
        private final Object principal;
        // 주로 사용자의 password에 해당함
        private Object credentials;
        
        // 인증 완료 전의 객체 생성
        public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
            super(null);
            this.principal = principal;
            this.credentials = credentials;
            setAuthenticated(false);
        }
        
        // 인증 완료 후의 객체 생성
        public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
                Collection<? extends GrantedAuthority> authorities) {
            super(authorities);
            this.principal = principal;
            this.credentials = credentials;
            super.setAuthenticated(true); // must use super, as we override
        }
    }
```
* AuthenticationProvider
    - 실제 인증에 대한 부분을 처리, 인증 전의 Authentication 객체를 받아 인증이 완료된 객체를 반환하는 역할

* AuthenticationManager
    - 인증에 대한 부분을 처리, AuthenticationManager에 등록된 AuthenticationProvider가 처리
    - 인증이 성공하면 `isAuthenticated == true`인 객체를 생성하여 SecurityContext에 저장
    - 실패 시 `AuthenticationException` 발생
    - AbstractUserDeatailsAuthenticationProvider 를 상속받아 DaoAuthenticationProvider가 실제 인증 과정에 대한 로직 처리
    - WebSecurityConfigurerAdapter를 상속받은 ApplicationSecurityConfig에서 AuthenticationManager에 DaoAuthenticationProvider 등록

```java
    @Configuration
    @EnableWebSecurity
    public class ApplicationSecurityConfig extends WebSecurityConfigurerAdapter {

        private final PasswordEncoder passwordEncoder;

        @Autowired
        public ApplicationSecurityConfig(PasswordEncoder passwordEncoder) {
            this.passwordEncoder = passwordEncoder;
        }

        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            auth.authenticationProvider(daoAuthenticationProvider());
        }

        @Bean
        public DaoAuthenticationProvider daoAuthenticationProvider() {
            DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
            provider.setPasswordEncoder(passwordEncoder);
            provider.setUserDetailsService(applicationUserService);

            return provider;
        }
    }
```

* UserDetails
    - 인증에 성공하여 생성된 UserDetails 객체는 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다,.
    - UserDetails 인터페이스의 경우 직접 개발한 ApplicationUser에 UserDetails를 상송받으면 된다.
    
* UserDetailsService
    - UserDetails 객체를 반환하는 단 하나의 메소드를 가지고 있는대, 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB에 연결하여 처리
    
* GrantedAuthority
    - 현재 사용자가 가지고 있는 권한들
    - ROLE_ADMIN, ROLE_USER 형태로 사용


### 시큐리티 & JWT
    - 스프링 시큐리티(세션기반)는 세션 설정을 해제하고, API 서버를 사용하기 위해서는 CSRF 보안도 필요 없다. 

### Ref.
* <https://sjh836.tistory.com/165>
* <https://sieunlim.tistory.com/19>
* <https://mangkyu.tistory.com/57>
