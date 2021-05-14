---
layout: post
title:  "Spring-servlet"
date:   2021-05-15 23:15
categories: spring,http
tags: [spring, http]
---
### Servlet

* 김영한 선생님 강의에서 배운것 - 3

> JSON
* JSON 결과를 파싱해서 사용할 수 있는 자바 객체로 변환하려면 Jackson, Gson같은 Json 변환 라이브러리를 추가해야함
* 스프링부트로 mvc를 선택한다면 기본적으로 Jackson 라이브러리(`ObjectMapper`)를 함께 제공
* HTML form 데이터도 메시지 바디를 통해 전송되므로 직접 읽을 수 있음.

> HttpServletResponse

* HTTP 응답 메시지 생성
    - 응답코드 지정

    - 헤더생성

    - 바디생성

    - ContentType, 쿠키, Redirect
    
```java

@WebServlet(name = "responseHeaderServlet", urlPatterns = "/response-header")
public class ResponseHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        // status - line
        response.setStatus(HttpServletResponse.SC_OK);

        // response - header
        response.setHeader("Content-Type", "text/plain");
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate"); // 캐시 완전 무효화
        response.setHeader("Pargma", "no-cache"); // 과거 버전의 캐시 무효화
        response.setHeader("my-header", "hello"); //

        //cookie
        cookie(response);
        
        // redirect
        redirect(response);

        PrintWriter printWriter = response.getWriter();
        printWriter.write("ok");
    }

    private void cookie(HttpServletResponse response) {
        //Set-Cookie: myCookie=good; Max-Age=600; //
        response.setHeader("Set-Cookie", "myCookie=good; Max-Age=600"); 
        Cookie cookie = new Cookie("myCookie", "good"); 
        cookie.setMaxAge(600); //600초
        response.addCookie(cookie); 
    }

    private void redirect(HttpServletResponse response) throws IOException { //Status Code 302

        //response.setStatus(HttpServletResponse.SC_FOUND); //302
        //response.setHeader("Location", "/basic/hello-form.html");
        response.sendRedirect("/basic/hello-form.html");
    }
}

```

* json 응답

```java

@WebServlet(urlPatterns = "/json-response")
public class ResponseJsonServlet extends HttpServlet {

    private ObjectMapper mapper = new ObjectMapper();
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("application/json");
        request.setCharacterEncoding("utf-8");

        HelloData data = new HelloData();
        data.setUsername("hwang");
        data.setAge(28);

        response.getWriter().write(mapper.writeValueAsString(data));
    }
}

```

### Ref.
* <https://https://www.inflearn.com/>
