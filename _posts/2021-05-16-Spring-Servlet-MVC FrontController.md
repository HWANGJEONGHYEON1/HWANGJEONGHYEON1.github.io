---
layout: post
title:  "Spring-FrontController"
date:   2021-05-16 14:15
categories: spring,http
tags: [spring]
---
### FrontController

* 김영한 선생님 강의에서 배운것 - 5
    - JSP에서 MVC 리팩토링 과정을 공부했다.

* 프론트 컨트롤러
    - 프론트 컨트롤러는 서블릿 하나로 클라이언트의 요청을 받음
    - 요청에 맞는 컨트롤러를 찾아 호출
    - `입구`는 하나다.
    - 공통 처리가능
    - 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨
    - 스프링 MVC `DispatcherSevlet` 
#### v1
    - 구현할 컨트롤러들의 인터페이스를 만든다.
```java
public interface ControllerV1 {

    void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;

}

```
    - 모든 요청은 서블릿 하나가 다 받고 MAP으로 경로와 클래스를 저장
```java
@WebServlet(urlPatterns = "/front-controller/v1/*")
public class FrontControllerServletV1 extends HttpServlet {

    protected Map<String, ControllerV1> controllerMap = new HashMap<>();

    public FrontControllerServletV1() {
        controllerMap.put("/front-controller/v1/members/new-form",
                new MemberFormControllerV1());
        controllerMap.put("/front-controller/v1/members/save",
                new MemberSaveControllerV1());
        controllerMap.put("/front-controller/v1/members",
                new MemberListControllerV1());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("FrontControllerServletV1.service");

        String requestURI = req.getRequestURI();
        final ControllerV1 controller = controllerMap.get(requestURI);
        if (controller == null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        controller.process(req, resp);

    }
}

```
    * 각자 화면에 처리할 컨트롤러들을 구현    
```java
public class MemberFormControllerV1 implements ControllerV1 {

    @Override
    public void process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String viewPath = "/WEB-INF/views/new-form.jsp";
        final RequestDispatcher requestDispatcher = request.getRequestDispatcher(viewPath);
        requestDispatcher.forward(request, response);
    }
}

```

#### v2
    - http 요청
    - frontController 매핑정보 확인
    - 컨트롤러 호출
    - myView 반환
    - render() 호출
    - jsp forward
    - 응답

* 각 컨트롤러는 뷰를 반환해야함 -> 인터페이스 구현
```java
public interface ControllerV2 {

    MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;
}
```

* 모든 요청을 FrontController에서 받음
```java
@WebServlet(urlPatterns = "/front-controller/v2/*")
public class FrontControllerServletV2 extends HttpServlet {

    protected Map<String, ControllerV2> controllerMap = new HashMap<>();

    public FrontControllerServletV2() {
        controllerMap.put("/front-controller/v2/members/new-form",
                new MemberFormControllerV2());
        controllerMap.put("/front-controller/v2/members/save",
                new MemberSaveControllerV2());
        controllerMap.put("/front-controller/v2/members",
                new MemberListControllerV2());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("FrontControllerServletV2.service");

        String requestURI = req.getRequestURI();
        final ControllerV2 controller = controllerMap.get(requestURI);
        if (controller == null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        final MyView view = controller.process(req, resp);
        view.render(req, resp);

    }
}
```

* 각 경로로 호출받은 Controller는 View를 리턴
```java
public class MemberSaveControllerV2 implements ControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    public MyView process(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(age, username);
        memberRepository.save(member);

        request.setAttribute("member", member);
        return new MyView("/WEB-INF/views/save-result.jsp");
    }
}
```

#### v3
    - 서블릿 종속성 제거
        - HttpServletRequest request, HttpServletResponse response가 필요한 것인가?
        - 서블릿 기술을 몰라도 동작가능하게 만듬
        - 사용하지 않는 코드 제거
    - 컨트롤러에서 뷰 이름이 중복
    - 컨트롤러는 뷰의 논리 이름을 반환, 위치의 이름은 프론트 컨트롤러에서 처리
    - 향후 뷰의 폴더 위치가 함께 이동해도 프론트 컨트롤러만 고치면됨
        - /WEB-INF/views/new-form.jsp -> new-form 
        - /WEB-INF/views/save-result.jsp -> save-result
    - 흐름
        - 요청 -> 프론트 컨트롤러 -> 매핑 정보
        - 컨트롤러 호출 -> ModelView 반환
        - viewResolver 호출 -> myView 반환
        - render() 호출
        - myView -> html 응답

* 구현할 인터페이스 
```java
public interface ControllerV3 {

    ModelView process(Map<String, String> paramMap);

}
```

* FrontController
    - 컨트롤러에서만 필요한 응답 요청 객체를 받는다
    - 요청 온 데이터를 Map에 담는다
    - ModelView 객체에 Map을 넘겨준다.
    - myView에 modelView, request, response 넘겨 view 호출
```java
@WebServlet(urlPatterns = "/front-controller/v3/*")
public class FrontControllerServletV3 extends HttpServlet {

    protected Map<String, ControllerV3> controllerMap = new HashMap<>();

    public FrontControllerServletV3() {
        controllerMap.put("/front-controller/v3/members/new-form",
                new MemberFormControllerV3());
        controllerMap.put("/front-controller/v3/members/save",
                new MemberSaveControllerV3());
        controllerMap.put("/front-controller/v3/members",
                new MemberListControllerV3());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("FrontControllerServletV3.service");

        String requestURI = req.getRequestURI();
        final ControllerV3 controller = controllerMap.get(requestURI);
        if (controller == null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = new HashMap<>();
        Collections.list(req.getParameterNames()).forEach(p -> paramMap.put(p, req.getParameter(p)));

        ModelView mv = controller.process(paramMap);

        String viewName = mv.getViewName();
        final MyView view = viewResolver(viewName);

        view.render(mv.getModel(), req, resp);

    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}

```

* ModelView
```java
@Getter
@Setter
public class ModelView {

    private String viewName;
    private Map<String, Object> model = new HashMap<>();

    public ModelView(String viewName) {
        this.viewName = viewName;
    }
}
```

* MyView 
```java
public class MyView {

    private String viewPath;

    public MyView(String viewPath) {
        this.viewPath = viewPath;
    }

    public void render(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    public void render(Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        ModelToRequestAttribute(model, request);
        RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
        dispatcher.forward(request, response);
    }

    private void ModelToRequestAttribute(Map<String, Object> model, HttpServletRequest request) {
        model.forEach((key, value) -> {
            request.setAttribute(key, value);
        });
    }
}
```

#### v4
    - v3 -> 실용성있는 버전으로
    - 모델 객체 전달
        - `Map<String, Object> model = new HashMap<String,Obeject>();`
        - 모델 객체를 프론트 컨트롤러에서 생성해서 넘겨준다.

* ControllerInterface
```java
public interface ControllerV4 {

    String process(Map<String, String> paramMap, Map<String, Object> model);

}
```

* FrontController
```java

@WebServlet(urlPatterns = "/front-controller/v4/*")
public class FrontControllerServletV4 extends HttpServlet {

    protected Map<String, ControllerV4> controllerMap = new HashMap<>();

    public FrontControllerServletV4() {
        controllerMap.put("/front-controller/v4/members/new-form",
                new MemberFormControllerV4());
        controllerMap.put("/front-controller/v4/members/save",
                new MemberSaveControllerV4());
        controllerMap.put("/front-controller/v4/members",
                new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("FrontControllerServletV3.service");

        String requestURI = req.getRequestURI();
        final ControllerV4 controller = controllerMap.get(requestURI);
        if (controller == null) {
            resp.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        Map<String, String> paramMap = new HashMap<>();
        Map<String, Object> model = new HashMap<>();

        Collections.list(req.getParameterNames()).forEach(p -> paramMap.put(p, req.getParameter(p)));

        String viewName = controller.process(paramMap, model);

        final MyView view = viewResolver(viewName);

        view.render(model, req, resp);

    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}

```

* controller interface 구현체
```java
public class MemberSaveControllerV4 implements ControllerV4 {

    final MemberRepository repository = MemberRepository.getInstance();

    @Override
    public String process(Map<String, String> paramMap, Map<String, Object> model) {
        String username = paramMap.get("username");
        int age = Integer.parseInt(paramMap.get("age"));

        Member member = new Member(age, username);
        repository.save(member);
        model.put("member", member);
        return "save-result";
    }
}
```

#### v5 
    - 개발자 마다 스타일이 달라 원하는 v3, v2 로 원한다면 어떻게 해야할지
        - `어뎁터 페턴`
    - 핸들러 어뎁터
        - 중간에 어뎁터 역할을 하는 어댑터 추가, 덕분에 여러 종류의 컨트롤러를 호출가능

* 사용할 어뎁터 인터페이스

```java
public interface MyHandlerAdapter {

    boolean supports(Object handler);

    ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException;
}

```
    
* V3 버전 어뎁터

```java
public class ControllerV3HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV3);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        final ControllerV3 controller = (ControllerV3) handler;
        Map<String, String> paramMap = createParamMap(request);
        final ModelView mv = controller.process(paramMap);
        return mv;
    }

    private Map<String, String> createParamMap(HttpServletRequest req) {
        Map<String, String> paramMap = new HashMap<>();
        Collections.list(req.getParameterNames()).forEach(p -> paramMap.put(p, req.getParameter(p)));
        return paramMap;
    }

}
```

* v4 버전
```java

public class ControllerV4HandlerAdapter implements MyHandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return (handler instanceof ControllerV4);
    }

    @Override
    public ModelView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws ServletException, IOException {
        ControllerV4 controller = (ControllerV4) handler;
        Map<String, String> paramMap = createParamMap(request);
        HashMap<String, Object> model = new HashMap<>();

        System.out.println(model);
        String viewName = controller.process(paramMap, model);
        System.out.println(model);
        ModelView modelView = new ModelView(viewName);
        modelView.setModel(model);

        return modelView;
    }

    private Map<String, String> createParamMap(HttpServletRequest req) {
        Map<String, String> paramMap = new HashMap<>();
        Collections.list(req.getParameterNames()).forEach(p -> paramMap.put(p, req.getParameter(p)));
        return paramMap;
    }
}
```

* FrontController

```java

@WebServlet(urlPatterns = "/front-controller/v5/*")
public class FrontControllerServletV5 extends HttpServlet {

    private final Map<String, Object> handlerMappingMap = new HashMap<>();
    private final List<MyHandlerAdapter> handlerAdapters = new ArrayList<>();

    public FrontControllerServletV5() {
        initHandlerMappingMap();
        initHandlerAdapters();
    }

    private void initHandlerAdapters() {
        handlerAdapters.add(new ControllerV3HandlerAdapter());
        handlerAdapters.add(new ControllerV4HandlerAdapter());
    }

    private void initHandlerMappingMap() {
        handlerMappingMap.put("/front-controller/v5/v3/members/new-form",
                new MemberFormControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members/save",
                new MemberSaveControllerV3());
        handlerMappingMap.put("/front-controller/v5/v3/members",
                new MemberListControllerV3());

        handlerMappingMap.put("/front-controller/v5/v4/members/new-form",
                new MemberFormControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members/save",
                new MemberSaveControllerV4());
        handlerMappingMap.put("/front-controller/v5/v4/members",
                new MemberListControllerV4());
    }

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        Object handler = getHandler(request);

        if (handler == null) {
            response.setStatus(HttpServletResponse.SC_NOT_FOUND);
            return;
        }

        MyHandlerAdapter adapter = getHandlerAdapter(handler);
        ModelView mv = adapter.handle(request, response, handler);

        String viewName = mv.getViewName();
        final MyView view = viewResolver(viewName);

        view.render(mv.getModel(), request, response);
    }

    private MyHandlerAdapter getHandlerAdapter(Object handler) {

        for (MyHandlerAdapter adapter : handlerAdapters) {
            if (adapter.supports(handler)) {
                return adapter;
            }
        }
        throw new IllegalArgumentException("handler adapter not exist! " + handler);
    }

    private Object getHandler(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        return handlerMappingMap.get(requestURI);
    }

    private MyView viewResolver(String viewName) {
        return new MyView("/WEB-INF/views/" + viewName + ".jsp");
    }
}
```

> 정리
* v1 : 프론트 컨트롤러 도입
* v2 : view 분류
    - 반복되는 뷰 로직 분리
* v3 : Model 추가
    - 서블릿 종속성제거 (HttpRequest, HttpResponse)
    - 뷰 이름 중복 제거
* v4 : 구현 입장에서 ModelView를 직접 생성해서 바환하지 않도록 편리한 인터페이스 제공
* 어댑터 도입, 프레임워크를 확장성 있고 유연하게 설계


### Ref.
* <https://https://www.inflearn.com/>

