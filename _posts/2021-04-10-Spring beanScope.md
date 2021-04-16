---
layout: post
title:  "스프링 빈 스코프"
date:   2021-04-10 19:22:21 +0900
categories: spring
---
## Bean Scope
 - 빈이 존재할 수 있는 범위
 - 스프링은 다양한 스코프를 지원
 - ```싱글톤``` : 디폴트, 스프링 컨테이너 시작과 종료까지 유지되는 가장 넓은 범위
    - 빈을 조회하면 항상 같은 인스턴스의 스프링 Bean을 반환
    ```java
    @Test
    void singletonTest() {
        AnnotationConfigApplicationContext ac  = new AnnotationConfigApplicationContext(SingletonBean.class);

        final SingletonBean bean1 = ac.getBean(SingletonBean.class);
        final SingletonBean bean2 = ac.getBean(SingletonBean.class);
        System.out.println("bean1 = " + bean1);
        System.out.println("bean2 = " + bean2);
        assertThat(bean1).isSameAs(bean2);
        ac.close();
    }

    @Scope("singleton")
    static class SingletonBean {
      @PostConstruct
      public void init() {
          System.out.println("Singleton init");
      }

      @PreDestroy
        public void destroy() {
          System.out.println("Singleton destroy");
      }
    }
    ```
    
    ```
    결과 
    Singleton init
    bean1 = hello.core.scope.SingletonTest$SingletonBean@3b0fe47a
    bean2 = hello.core.scope.SingletonTest$SingletonBean@3b0fe47a
    Singleton destroy
    ```
  
 - ```프로토타입``` : 빈의 생성과 의존관계 주입, 초기화까지만 관여, 매우 짧은 범위
    - 스프링 컨테이너에 조회하면 항상 새로운 인스터스를 생성하여 반환
    ```java
    public class ProtoTypeTest {

        @Test
        void prototypeTest() {
            final AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
            final PrototypeBean bean1 = ac.getBean(PrototypeBean.class);
            final PrototypeBean bean2 = ac.getBean(PrototypeBean.class);
            System.out.println("bean1 = " + bean1);
            System.out.println("bean2 = " + bean2);
            assertThat(bean1).isNotEqualTo(bean2);
            ac.close();
        }

        @Scope("prototype")
        static class PrototypeBean {
            @PostConstruct
            public void init() {
                System.out.println("singleton init");
            }

            @PreDestroy
            public void destroy() {
                System.out.println("Singleton destroy");
            }
        }
    }
    ```

    ```
    실행결과
    singleton init
    singleton init
    bean1 = hello.core.scope.ProtoTypeTest$PrototypeBean@3b0fe47a
    bean2 = hello.core.scope.ProtoTypeTest$PrototypeBean@202b0582
    ```
    - 빈 생성이 새로 만들어지고 @PreDestroy가 호출되지 않는다.
    - 직접 bean2.destroy() 호출해줘야한다.
 - ```웹 관련 스코프```
    - ```request``` : 웹 요청이 오고 나갈때까지 유지
        - 동시에 여러 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 구분하기 어렵기때문에 request 스코프 사용하면 유용하다.
        - HTTP 요청 당 하나씩 생성, 요청이 끝나면 소멸
        - 
    - ```세션``` : 웹 세션이 생성되고 종료될때까지 유지
    - ```application``` : 웹의 서블릿 컨텍스트와 같은 범위

- 스코프와 프록시
  - ```@Scope(value= "request", proxyMode = ScopedProxyMode.TARGET_CLASS)```
  - CGLIB 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어주서 주입
  - 가짜 프록시 객체는 요청이오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 있음.
  - 내부에 실제 MyLogger를 찾는 방법을 알고 있다.
  - 동작 원리
    - CGLiB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어 주입
    - 실제 요청이 오면 내부에서 실제빈을 요청하는 위임로직이 들어있음
    - request scope와 상관없음, 내부에 단순한 위임로직만 있고, 싱글톤처럼 동작한다.
  - 특징 정리
    - 싱글톤처럼 쓰듯이 request scope를 사용할 수 있다.
    - Provider, 프록시의 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연시켜준다.
    - 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대채할 수 있다.
    - DI 컨테이너와 다형성
    - 웹 스코프가 아니어도 사용할 수 있다.
  - 주의점
    - 싱글톤처럼 사용하는 것 같지만 다르게 동작하기 때문에 주의
    - scope를 꼭 필요한 경우에만 사용해야한다.