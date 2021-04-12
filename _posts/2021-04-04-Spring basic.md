---
layout: post
title:  "IoC, DI, Container"
date:   2021-04-04 19:22:21 +0900
categories: spring
---

# 제어의 역전

> ```Inversion of Control```
- 기존의 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성, 연결, 실행하여 스스로 프로그램의 제어흐름을 가지고 있음
- 컨테이너가 등장함으로써 구현객체는 자신의 로직을 실행만 시킴
- 프로그램에 대한 제어권은 컨테이너가 가지고 있다. 
- ex)
```java
class AppConfig {

    public memberServive () {
        return new MemberServiceImpl(memberRepository);
    }

    public memberRepository() {
        return new OracleDbRepository(); // 원하는 디비를 연결하여 가져올 수 있다.
    }
}

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    // 어떤 repository를 받는지 신경안쓰고 일단 구현한다. (핵심이라고 생각)
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```
- 프레임워크 vs 라이브러리
  - 프레임워크 : 내가 작성한 코드를 제어하고, 대신 실행하면 프레임워크
  - 라이브러리 : 내가 작성한 코드가 직접 제어의 흐름을 담당하면 라이브러리

# 의존관계 주입

> Dependency Injection
- 정적인 클래스 의존관계
- 실행시점에 결정되는 동적인 객체 의존관계
  - 애플리케이션 _ 실행시점_ 에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달하여 클라이언트와 서버의 실제 의존관계가 연결되는 것을 ```의존관계주입```
  - 객체인스턴스를 생성하고, 그 참조값을 전달하여 연결
  - 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있음.
  - 정적인 클래스를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경


# IoC컨테이너, DI 컨테이너
 - 객체를 생성하고 관리해주는 컨테이너 위에 AppConfig.class 역할
 - IoC컨테이너 또는 DI 컨테이너
 - 의존관계 주입에 초점을 맞추어 DI컨테이너라고 불린다.