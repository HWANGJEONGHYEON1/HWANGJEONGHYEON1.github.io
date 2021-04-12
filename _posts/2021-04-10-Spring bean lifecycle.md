---
layout: post
title:  "스프링 빈 생명주기"
date:   2021-04-10 19:22:21 +0900
categories: spring
---

## Bean 생명주기 콜백
 - 객체생성 -> 의존관계주입
   - 스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에 필요한 데이터를 사용할 수 있는 준비가 완료된다.
 - 스프링 컨테이너 생성 -> 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸전 콜백 -> 스프링 종료


## 빈 생명주기 콜백 3가지
 - 인터페이스
   - InitializingBean, DisposableBean
   ```java 
    public class NetworkClient implements InitializingBean, DisposableBean { ... 
        @Override
        public void afterPropertiesSet() throws 
        Exception {
            connect();
        }
        @Override
        public void destroy() throws Exception {
            disConnect();
        }
   ```
   - 단점 : 스프링 인터페이스에 의존하여 초기화 소멸 메서드 이름 변경할 수 없다. 코드를 고칠 수 없는 외부 라이브러리 적용 불가 (현재 거의 사용되지 않음)
 - 설정 정보 초기화 메서드, 종료메서드
   ```java 
    @Bean(initMethod = "init", destroyMethod = "close")
   ```
   - 메서드 이름을 자유롭게 줄 수 있다.
   - 스프링 코드에 의존하지 않는다.
   - 외부라이브러리에 적용가능하다.

    
 - @PostConstruct, @PreDestroy 애노테이션
   ```java
   @PostConstruct, @PreDestory
   ````
   - 스프링에서 권장하는 방법
   - javax.annotation.PostConstruct -> 스프링 종속기술이 아니라 자바표준
   - 컴포넌스 스캔과 잘어울린다.
   - 외부 라이브러리를 초기화 종료 못한다 ->@Bean 기능사용(위)

### 정리 
 - @PostConstruct, @PreDestory 애노테이션을 사용하자
 - 코드를 고칠 수 없는 외부 라이브러리를 초기화, 종료해야 하면 @Bean 의 initMethod , destroyMethod 를 사용하자.