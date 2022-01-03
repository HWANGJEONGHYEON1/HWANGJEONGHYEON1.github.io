---
layout: post
title:  "Modern Java - 9"
date:   2022-01-03 11:20:21 +0900
categories: java
---

> 기본에 충실하자 - 9

# 리액티브 프로그래밍 

### 리액티브 매니패스토
- 반응성 : 리액티브 시스템은 빠르면서 일정하고 예상가능한 반응시간을 제공
- 회복성 
    - 장애가 발생하여도 시스템은 동작가능해야한다.
    - 각 컴포넌트가 비동기적으로 작업을 다른 컴포넌트에 위암하는 등 리액티브 매니페스토는 회복성을 달성할 수 있는 다양한 기법을 제공
- 탄령석 : 어플리케이션의 생명주기 동안 다양한 부하가 발생하는대, 무거운 작업부하가 발생할 경우 자동으로 컴포넌트에 할당된 지원 수를 늘린다.
- 메시지 주도 
    - 회복성과 탄령성을 지원하려면 약 결합, 고립, 위치 투명성등을 지원할 수 있도록 시스템을 구성하는 컴포넌트 경계를 명확히 정의해야한다.
    - 비동기메시지를 전달하여 컴포넌트끼리 통신이 이루어진다.


### 애플리케이션 수준의 리액티브
- 어플리케이션 수즌의 리액티브 프로그래밍의 주요기능은 비동기 작업을 수행할 수 있다는 것.
- 이벤트 스트림을 복사하지 않고 비동기로 처리하는 것이 최신 멀티코어 CPU의 사용률을 극대화
- 리액티브 프레임워크나 라이버르리는 스레드를 퓨처, 액터, 일련의 콜백을 발생시키는 이벤트 루프등과 공유하고 처리할 이 이벤트를 변환 및 관리
- 스레드를 다시 쪼개는 종류의 기술을 사용할 경우 메인 이벤트 루프 안에서는 절대 동작을 블락하지 않아야한다는 전제가 필요

### 시스템 수준의 리액티브
- 리액티브 시스템은 여러 어플리케이션이 한 개의 일관적이고 회복가능한 플랫폼을 구성할 수 있게 해줄 뿐아니라 이들 중 하나가 실패해도 전체 시스템은 운영될 수 있게 해주는 소프트웨어 아키텍처
- 리액티브 어플리케이션은 짧은 시간 동안 연산을 수행하지만 리액티브 시스템은 어플리케이션을 조립이라고 생각하고 상호소통을 조절
- 주요 속성은 메시지 주도를 꼽는다
- 수신, 발신 메시지 등과 같은 각각의 컴포넌트들이 서로 결합되지 않도록 구현해야한다. 그래야만 시스템이 장애와 높은 부하에도 반응셩을 유지할 수 있다.

### 리액티브 스트림과 FLOW API
- 리액티브 프로그래밍은 리액티브 스트림을 사용하는 프로그래밍이다. 리액티브 스트림은 잠재적으로 무한의 비동기 데이터를 순서대로 그리고 블록하지 않는 배압을 전제해 처리하는 표준 기술
- 스트림 처리의 비동기적은 특성상 배압의 기능의 내장은 필수이다. 실제 비 동기 작업이 실행되는 동안 시스템에는 암묵적으로 블록 API로 인해 배압이 제공되는것.

### FLOW class
- 자바 9 에서는 리액티브 프로그래밍을 제공하는 Flow를 추가
- 정적 컴포넌트 하나를 포함하고 있으며 인스턴스화 할 수 없다.
- Publisher가 항목을 발행하면 Subscriber가 한 개 또는 여러 항목을 소비하는데, Subscription이 이 과정을 관리할 수 있도록 FLow 클래스는 관련된 인터페이스와 정적 메서드를 제공한다.
- 발행 - 구독 모델을 지원할 수 있도록 Flow 클래스는 중첩된 인터페이스 네개를 포함
    - Publisher
    - Subscriber
    - Subscription
    - Processor

```java


    @FunctionalInterface
    public static interface Publisher<T> {
        /**
         * Adds the given Subscriber if possible.  If already
         * subscribed, or the attempt to subscribe fails due to policy
         * violations or errors, the Subscriber's {@code onError}
         * method is invoked with an {@link IllegalStateException}.
         * Otherwise, the Subscriber's {@code onSubscribe} method is
         * invoked with a new {@link Subscription}.  Subscribers may
         * enable receiving items by invoking the {@code request}
         * method of this Subscription, and may unsubscribe by
         * invoking its {@code cancel} method.
         *
         * @param subscriber the subscriber
         * @throws NullPointerException if subscriber is null
         */
        public void subscribe(Subscriber<? super T> subscriber);
    }


    public static interface Subscriber<T> {
        /**
         * Method invoked prior to invoking any other Subscriber
         * methods for the given Subscription. If this method throws
         * an exception, resulting behavior is not guaranteed, but may
         * cause the Subscription not to be established or to be cancelled.
         *
         * <p>Typically, implementations of this method invoke {@code
         * subscription.request} to enable receiving items.
         *
         * @param subscription a new subscription
         */
        public void onSubscribe(Subscription subscription);

        /**
         * Method invoked with a Subscription's next item.  If this
         * method throws an exception, resulting behavior is not
         * guaranteed, but may cause the Subscription to be cancelled.
         *
         * @param item the item
         */
        public void onNext(T item);

        /**
         * Method invoked upon an unrecoverable error encountered by a
         * Publisher or Subscription, after which no other Subscriber
         * methods are invoked by the Subscription.  If this method
         * itself throws an exception, resulting behavior is
         * undefined.
         *
         * @param throwable the exception
         */
        public void onError(Throwable throwable);

        /**
         * Method invoked when it is known that no additional
         * Subscriber method invocations will occur for a Subscription
         * that is not already terminated by error, after which no
         * other Subscriber methods are invoked by the Subscription.
         * If this method throws an exception, resulting behavior is
         * undefined.
         */
        public void onComplete();
    }

```

- Subscriber 이벤트는 다음에서 정의된 순서대로 발행되어야한다. 
    - onNext는 여러번 호출이 가능하다.

```
onSubscribe onNext* onError onComplete
```

- Subscription 클래스는 첫번째로 Publisher에게 주어진 개수 이벤트를 처리할 준비가 되었음을 알릴 수 있다.
- 두번째로는 Subscription을 취소, 더 이상 Publisher에게 이벤트에 대해 통지를 받지 않는다.
- java 9 flow docs에는 인터페이스 구현이 어떻게 서로 협력하는지 나와있다.
    - Publisher는 반드시 Subscriber의 request를 메서드에 정의된 개수 이하의 요소만 Subscription에게 전달한다.
    - Subscriber는 요소를 받아 처리할 수 있음을 Publisher에게 알려야한다.
    - Publisher와 Subscriber는 정확하게 Subscription을 공유해야하며, 각각이 고유한 역할을 수행해야한다. 그러기 위해 onSubscriber와 onNext메서드에서 Subscriber는 request 메서드를 동기적으로 호출할 수 있어야한다.
- FLow 클래스의 네번째 Processor 인터페이스는 단지 Publisher와 Subscriber를 상속받을 뿐 아니라 아무 메서드도 추가하지 않는다.

```java

    public static interface Subscription {
        /**
         * Adds the given number {@code n} of items to the current
         * unfulfilled demand for this subscription.  If {@code n} is
         * less than or equal to zero, the Subscriber will receive an
         * {@code onError} signal with an {@link
         * IllegalArgumentException} argument.  Otherwise, the
         * Subscriber will receive up to {@code n} additional {@code
         * onNext} invocations (or fewer if terminated).
         *
         * @param n the increment of demand; a value of {@code
         * Long.MAX_VALUE} may be considered as effectively unbounded
         */
        public void request(long n);

        /**
         * Causes the Subscriber to (eventually) stop receiving
         * messages.  Implementation is best-effort -- additional
         * messages may be received after invoking this method.
         * A cancelled subscription need not ever receive an
         * {@code onComplete} or {@code onError} signal.
         */
        public void cancel();
    }

    public interface Processor<T, R> extends Subscriber<T>, Publisher<R> { }


```