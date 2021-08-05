---
layout: post
title:  "리액트프로그래밍"
date:   2021-06-29 15:00
categories: book, jpa
tags: [spring. reactprogramming]
---

## 리액트 프로그래밍
> 변화의 전파와 데이터 흐름과 관련된 선언적 프로그래밍 패러다임.

### 변화의 전파와 데이터 흐름 
- 데이터가 변경될 때마다 이벤트를 발생시켜서 데이터를 지속적으로 전달

### 리액티브 프로그래밍
- Observable : 데이터 소스
- 리액티브 연산자(Operators) : 데이터 소스를 처리해주는 함수
- 스케쥴러 : 스레드 관리자
- Subscriber : Observable이 발행하는 데이터를 구독하는 구독자
- 함수형 프로그래밍 : RxJava 에서 제공하는 연산자 함수를 사용

```java


        // 데이터발행
        // 데이터 가공
        // 데이터 구독하여 처리
        Observable.just(100,200,300,400,500)
                .doOnNext(data -> System.out.println(getThreadName() + " : "  + " #doOnNext() " + data))
                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.computation()) // 데이터의 흐름을 처리
                .filter(num -> num > 300)
                .subscribe(num -> System.out.println(getThreadName() + " : result : " + num));

        Thread.sleep(500);

```

### 마블 다이어그램
- 리액티브 프로그래밍을 통해 발생하는 비동기적인 데이터의 흐름을 시간의 흐름에따라 시각적으로 표시한 다이어그램