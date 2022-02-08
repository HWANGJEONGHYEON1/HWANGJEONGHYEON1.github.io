---
layout: post
title:  "빈 후처리기"
date:   2022-02-08 07:21:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근) 10번 째

# 빈 후처리기
> @Bean이나 컴포넌트 스캔으로 스프링 빈을 등록하면, 스프링은 대상 객체를 생성하고 스프링 컨테이너 내부에의 빈 저장소에 등록을 한다. 그리고 이후에는 스프링 컨테이너를 통해 등록한 스프링 빈을 조회해서 사용하면 된다.
- 빈을 생성한 후에 무언가를 처리할 용도로 사용
- 객체 조작
- 다른 객체로 변경 가능

### BeanPostProcessor
- 빈을 조작하고 변경할 수 있는 후킹 포인트
- 빈객체를 조작하거나 다른 객체로 바꾸어 버릴 수 있을 정도로 막강하다.
- 개발자가 등록하는 모든 빈을 중간에 조작할 수 있다. => 빈 객체를 `프록시`로 변경하는 것도 가능하다. 
- 빈 후처리기를 사용하려면 `BeanPostProcessor`을 구현하고, 스프링빈을 등록하면 된다.
- beanA를 스프링 빈으로 등록했지만, 빈 후처리기를 통해 빈A를 빈B로 바꿔치기한다.
- 결국 A 대신에 B가 등록되어 beanA는 객체로 등록되지 않는다.
- 코드

```java
@Test
    void basicConfig() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanPostProcessorConfig.class);
        B beanB = applicationContext.getBean("beanA", B.class);
        beanB.helloB();

        // A 빈은 등록되지 않음
        Assertions.assertThrows(NoSuchBeanDefinitionException.class, () -> applicationContext.getBean(A.class));
    }

    static class A {
        public void helloA() {
            log.info("Hello A");
        }
    }

    static class B {
        public void helloB() {
            log.info("Hello B");
        }
    }

    static class BeanPostProcessorConfig {
        @Bean(name = "beanA")
        public A a() {
            return new A();
        }

        @Bean
        public AtoBPostProcessor helloPostProcessor() {
            return new AtoBPostProcessor();
        }
    }

    static class AtoBPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            log.info("beanName={}, bean={}", beanName, bean);

            if (bean instanceof  A) {
                return new B();
            }
            return bean;
        }
    }
```

### 스프링이 제공하는 빈 후처리기
> implementation 'org.springframework.boot:spring-boot-starter-aop' 추가
- 자동 프록시 생성기 활성화 => AutoProxtyCreator
- AnnotationAwareAspectJAutoProxyCreator 라는 빈 후처리기가 스프링 빈에 자동으로 등록된다.
- 이 빈 후처리기는 `Advisor`들을 자동으로 찾아서 프록시가 필요한 곳에 자동으로 프록시를 적용해준다.
    - Advisor 안에는 Advice, Pointcut이 있어서 어떤 스프링 빈에 프록시를 적용해야할지 알 수 있다.
- @AspectJ도 등록해준다.
- `포인트 컷` 2가지 사용
    - 프록시 적용 여부 판단 - 생성 단계
        - 자동 프록시 생성기는 포인트컷을 사용해서 해당 빈이 프록시를 생성할 필요가 있는지 없는지 체크
        - 클래스 + 메서드 조건을 모두 비교. 모든 메서드를 체크하는데, 포인트컷 조건에 하나하나 매칭. 조건에 맞는것이 하나라도 있으면 프록시 생성
    - 어드바이스 적용 여부 판단 - 사용 단계
        - 프록시가 호출되었을 떄 부가기능인 어드바이스를 적용할지 말지 포인트 컷을 보고 판단.
