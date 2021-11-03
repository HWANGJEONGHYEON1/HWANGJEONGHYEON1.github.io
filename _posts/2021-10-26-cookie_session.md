---
layout: post
title: "쿠키 세션"
date:  2021-10-26 22:20
categories: spring, cookie, session
tags: [spring, cookie, session]
---

## 로그인 관련 개발을 할 때 쿠키, 세션 적용기

### COOKIE
```java
    @PostMapping("/login")
    public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        final Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
        log.info("loginMember {}", loginMember);

        if (loginMember == null) {
            log.info("error ");
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        // Login 성공 처리
        // 쿠키에 시간 정보를 주지 않으면 세션쿠키
        final Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
        response.addCookie(idCookie);

        return "redirect:/";
    }
```
- 이 소스에 치명적인 문제는 쿠키에 사용자 정보가 들어간다.
- 제 3자가 브라우저에서 직접 `조작이 가능`
- 심각한 보안 문제
- 대안
  - 쿠키에 중요한 값을 노출하지 않고, 사용자 별 예측 불가능한 임의의 `토큰`을 노출하여, 서버에서 토큰과 사용자 id를 매핑해서 인식
  - `서버`에서 토큰관리
  - 만료시간 유지

### SESSION
- 서버에서 중요한 정보를 보관하고 연결을 유지하는 방법
- 멤버와 관련된 정보는 클라이언트에 전달하지 않고 임의의 토큰을 전달
- 해커가 토큰을 가져가도 시간이지나면 사용할 수 없도록 세션 만료 시간을 짧게 유지하면 된다.
- 응답에 토큰에 대한 아이디를 쿠키에 저장하고
- 요청에는 쿠키에 저장한 value를 보내서 세션에서 조회한다.
```java

@Component
public class SessionManager {

    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();
    private static final String SESSION_COOKIE_NAME = "mySessionId";

    /**
     * 세션생성
     *  * sessionId 생성(추정 불가능한 임의의 값)
     *  * 세션 저장소에 sessionId를 보관할 값 저장
     *  * sessionId로 응답 쿠키를 생성해서 클라이언트에 저장
     */
    public void createSession(Object value, HttpServletResponse response) {

        final String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        final Cookie cookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(cookie);
    }

    public Object getSession(HttpServletRequest request) {
        final Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }

        return sessionStore.get(sessionCookie.getValue());
    }

    public void expire(HttpServletRequest request) {
        final Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        final Cookie[] cookies = request.getCookies();
        if (cookies == null) {\
            return null;
        }

        return Arrays.stream(cookies)
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }
}
```

### HttpSession
- 서블릿을 통해 HttpSession을 생성하면 `JSESSIONID` 쿠키를 생성하고 값은 추정불가능한 랜덤값이 된다.

```java
        // session이 있으면 세션반환, 없으면 생성
        final HttpSession session = request.getSession();
        // 세션에 로그인 회원 정보를 보관
        session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);

    // 세션 정보
    @GetMapping("/session-info")
    public String sessionInfo(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session == null) {
            return "Session is not exist";
        }
        // 세션 데이터 출력
        session.getAttributeNames()
                .asIterator()
                .forEachRemaining(name -> log.info("session Name = {}, value = {}", name, session.getAttribute(name)));
        log.info("sessionId={}", session.getId());
        log.info("getMaxInactiveInterval()={}", session.getMaxInactiveInterval());
        log.info("getCreationTime()={}", new Date(session.getCreationTime()));
        log.info("getLastAccessedTime()={}", new Date(session.getLastAccessedTime()));
        log.info("isNew={}", session.isNew());
        return "세션 출력";
    }
```

- 세션 타임아웃 설정(로그인 시 얼마 후에 세션 삭제)
  - 세션 타임아웃 시간은 해당 세션과 관련된 JSESSIONID를 전달하는 요청이 있으면 현재시간으로 다시 초기화(`getLastAccessedTime`)
  - getLastAccessedTime -> 이후로 시간이 지나면 was가 내부에서 세션을 삭제한다.
> server.servlet.session.timeout=60


