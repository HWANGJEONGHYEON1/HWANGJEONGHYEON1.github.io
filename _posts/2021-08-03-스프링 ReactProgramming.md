---
layout: post
title:  "리액트프로그래밍"
date:   2021-08-02 18:20
categories: book, webflux
tags: [spring-webflux, reactprogramming]
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
                .observeOn(Schedulergs.computation()) // 데이터의 흐름을 처리
                .filter(num -> num > 300)
                .subscribe(num -> System.out.println(getThreadName() + " : result : " + num));

        Thread.sleep(500);

```

### 마블 다이어그램
- 리액티브 프로그래밍을 통해 발생하는 비동기적인 데이터의 흐름을 시간의 흐름에따라 시각적으로 표시한 다이어그램

### 리액티브 스트림(Reactive Streams)
- 리액티브 프로그래밍 라이브러리의 표준 사양
- 인터페이스만 제공
- Publisher
    - 데이터를 생성하고 통지
- Subscriber
    - 통지된 데이터를 전달받아 처리
- Subscription
    - 전달받을 데이터의 개수를 요청하고 구독을 해지
- Processor
    - Publisher와 Subscriber의 기능이 모두 있음.


### Cold Publisher , Hot Publisher
- Cold Publisher
    - 생산자는 소비자가 구독할 때마다 데이터를 처음부터 새로 통지한다.
    - 데이터를 통지하는 새로운 타임라인이 생성된다.
    - 소비자는 구독시점과 상관없이 통지된데이터를 처음부터 전달 받을 수 있다.
- Hot Publisher
    - 생산자는 소비자 수와 상관없이 데이터를 한번만 통지한다.
    - 데이터를 통지하는 타임라인은 하나
    - 소비자는 발행된 데이터를 처음부터 전달받는게 아니라 구독 시점에 통지된 데이터를 받을 수 있다.

###  Flowable vs Observable 
- Flowable
    - Reactive Streams 인터페이스를 구현
    - Subscriber에서 데이터를 처리
    - 데이터 개수제어하는 `배압`기능이 있음
    - Subscription으로 전달받은 데이터 개수를 제어할 수 있음
    - Subscription으로 구독 해지
- Observable
    - Reactive Streams 인터페이스를 구현 X
    - Observer에서 데이터 처리
    - `배압`기능 없음
    - 데이터 개수를 제어할 수 없음
    - Disposable로 구독 해지

### 배압이란
    - Flowable에서 데이터를 통지하는 속도가 Subscriber에서 통지된 데이터를 전달받아 처리하는 속도보다 빠를 때 밸런스를 맞추기위해 데이터 통지량을 제어하는 기능
    - Missing 전략
        - 배압을 적용하지 않는다.
        - onBackpressureXXX()로 배압적용을 할 수 있다.
    - ERROR 전략
        - 통지된 데이터가 버퍼의 크기를 초과하면 MissingBackPressure 에러를 통지한다.
        - 즉 소비자가 생산자의 통지속도를 따라잡지 못할 때 발생
    - 버퍼 전략 : DROP_LASTEST
        - 버퍼가 가득 찬 시점에 버퍼내에서 가장 최근에 버퍼로 들어온 데이터를 드랍
        - 드랍 된 빈 자리에 버퍼 밖에서 대기하던 데이터를 채운다.   

### Single
- 데이터를 1건만 통지하거나 에러를 통지
- 데이터 통지 자체가 이미 완료되었으므로, 완료를 통지않음
- 데이터를 1건만 통지하므로 데이터 개수를 요청하지 않음
- onNext(), onComplete()를 합한 onSuccess()를 제공
- 클라이언트의 요청에 대한 응답이 대표적인 Single을 사용
- singleObserver 소비자

```java
 Single.just(DateUtil.getNowDate())
                .subscribe(
                        data -> System.out.println(data),
                        error -> System.out.println(error)
                );
```


### Maybe
- 데이터를 1건만 통지하거나, 1건도 통지하지 않고 에러를 통지
- 데이터 통지 자체가 완료를 의미하지 않기때문에 완료를 통지하지 않는다.
- 데이터를 1건도 통지하지않고 처리될 경우, 완료 통지를 한다.
- MaybeObserver 

```java
Single<String> single = Single.just(DateUtil.getNowDate());
        Maybe.fromSingle(single)
                .subscribe(
                        data -> Logger.log(LogType.ON_SUCCESS, "# 현재 날짜시각: " + data),
                        error -> Logger.log(LogType.ON_ERROR, error),
                        () -> Logger.log(LogType.ON_COMPLETE)
                );
```

### Completable
- 데이터 생산자이지만 데이터를 통지하지않고, 완료 또는 에러를 통지
- 데이터 통지의 역할 대신에 Completable 내에서 특정 작업을 수행한 후, 해당 처리가 끝났음을 통지하는 역할을 한다.
- Completable의 대표적인 소비자는 CompletableObserver이다. 

