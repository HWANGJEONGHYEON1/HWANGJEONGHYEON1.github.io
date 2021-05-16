---
layout: post
title:  "Spring-Servlet-MVC"
date:   2021-05-16 22:15
categories: spring,http
tags: [spring, http, servlet]
---
### Servlet

* 김영한 선생님 강의에서 배운것 - 4

> 템플릿 엔진으로
* 서블릿으로 작성한다면 자바 코드로 HTML을 만들어 내는 것은 매우 비효율적
* HTML 문서에 동적으로 변경해야하는 부분만 자바코드로 넣는 것이 편하여 템플릿 엔진이 나왔다
* HTML에서 필요한 곳만 코드를 동적으로 변경

> MVC 패턴 등장
* 비즈니스 로직은 서블릿처럼 다른 곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면을 그리는 일에 집중
* 많은 고민이 있었고 그러므로, MVC 패턴 등장


> MVC 패턴 - 개요
    - 기능 특화
        - JSP 같은 뷰 템플릿은 화면을 렌더링하는데 최적화 되어 있기때문에 이 부분의 업무만 담당하는 것이 효과적
        - Model View Controller
            - Controller : HTTP의 요청을받아 파라미터를 검증하고, 비즈니스로직을 실행. 그리고 뷰에 전달할 결과 데이터를 조회하여 데이터를 모델에 담는다
            - Model : 뷰에 출력할 데이터르 담는다. 비즈니스 로직이나 데이터 접근을 몰라도되고, 화면을 렌더링에 집중
            - View : 모델에 담겨있는 데이터를 사용하여 화면을 그리는 일에 집중.

> redirect vs foward
    - redirect
        - 리다이렉트는 실제 브라우저에 응답이 나갔다가, 클라이언트가 redirect 경로로 다시 요청. 따라서 클라이언트가 인지할 수 있고, 실제 경로도 변경된다.
        - URL 경로도 실제로 변경
    - foward
        - 서버 내부에서 일어나는 호출이기때문에 클라이언트가 전혀 인지하지 못한다.

* 코드 (save, list)

```java
// 회원가입화면으로 foward
@WebServlet(urlPatterns = "/servlet-mvc/members/new-form")
public class MvcMemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        final RequestDispatcher requestDispatcher = request.getRequestDispatcher(viewPath);
        requestDispatcher.forward(request, response);
    }
}
```

```java
// 회원가입화면에서 저장 버튼
@WebServlet(urlPatterns = "/servlet-mvc/members/save")
public class MvcMemberSaveServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(age, username);
        memberRepository.save(member);

        request.setAttribute("member", member);
        String viewPath = "/WEB-INF/views/save-result.jsp";

        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }
}
```

```java
// 회원가입 저장 후 리스트 볼 수 있는 화면 foward
@WebServlet(urlPatterns = "/servlet-mvc/members")
public class MvcMemberListServlet extends HttpServlet {
    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        List<Member> memberList = memberRepository.findAll();

        request.setAttribute("members", memberList);

        RequestDispatcher dispatcher = request.getRequestDispatcher("/WEB-INF/views/members.jsp");
        dispatcher.forward(request, response);
    }
}
```

* 기본적으로 뷰는 컨트롤러를 통해 나가야한다.
* 기존 MVC 방식을 지키지 않은 방식은 Servlet 코드안에서 HTML코드도 작성하고 자바코드도 작성해야하기 떄문에 보기도 힘들다.
* 보기힘들다 라는 것은 유지보수하기 힘들고, 코드 줄이 방대해진다 라는 생각이든다.

> MVC 패턴의 한계
* 컨트롤러는 중복이 많고, 필요하지 않은 코드들이 있다.
` RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath); dispatcher.forward(request, response); `
`String viewPath = "/WEB-INF/views/new-form.jsp";`
* 공통처리가 어렵다.
    - 해결하기 위해서는 `수문장` 역할이 필요, `프론트 컨트롤러 패턴` 도입하면 된다.
    - 스프링 MVC 핵심도 이 프론트 컨트롤러로 해결하면 된다.




### Ref.
* <https://https://www.inflearn.com/>
