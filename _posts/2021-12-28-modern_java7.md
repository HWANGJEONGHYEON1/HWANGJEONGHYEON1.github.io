---
layout: post
title:  "Modern Java - 7"
date:   2021-12-28 16:20:21 +0900
categories: java
---

> 기본에 충실하자 - 7

# CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

### 동시성을 구현하는 자바 지원
- 자바는 Runnable과 Thread를 동기화된 클래스와 메서드를 이용하여 잠궜다.
- 그 후 ExceutorService 인터페이스, 높은 수준의 결과 Runnable, Thread를 변형하는 Callable과 Funture, 제네릭을 지원했다.
- ExceutorService는 Runnable과 Callable 둘다 실행 가능하다. 이런 기능이 추가됨에따라 멀티코어 CPU에서 쉽게 병렬 프로그램을 구현 할 수 있었다.
- 자바 5 : 스레드 실행과 테스트 제출을 분리하기 위한 Thread, Runnable의 변형 Callable, Future, 제네릭
- 자바 7 : 분할 정복 알고리즘 포크/조인 구현을 지원하는  java.util.concurrent.RecursiveTask
- 자바 8 : 스트림과 새로 추가된 람다, Future를 조합하는 기능을 추가하며 동시성을 강화한 CompletableFuture
- 자바 9 : 분산 비동기 프로그램을 지원, 리액티브 프로그래밍을 위한 Flow 인터페이스 추가

### 스레드와 높은 추상화
- 직접 스레드를 사용하지 않고 스트림을 이용하여 스레드 사용패턴을 추상화 할 수 있다.
- 스트림으로 추상화 하는 것은 디자인 패턴을 저용하는것과 비슷하지만 쓸모없는 코드가 라이브러리 내부로 구현되면서 복잡성이 줄어든다.

### Excutor와 스레드 풀
- Excutor 프레임워크와 스레드 풀을 통해 스레드를 테스크 제출과 실행을 분리할 수 있도록 구현할 수 있었다.
- 일반적으로 스레드를 생성하는 비용이 크기 때문에 적정의 갯수의 스레드를 스레드 풀에 미리 생성 등록하여 사용하였다
- 보통 운영체제와 자바의 스레드 수가 하드웨어 스레드 개수보다 많다. 프로그램에서 사용할 최적의 자바 스레드 개수는 사용할 수 있는 하드웨어 코어의 개수에 따라 달라진다.
- 프로그래머는 테스크(Runnable, Callable)를 제공하면 스레드가 이를 실행한다.
- 스레드 풀이 나쁜 이유
    - 모든 관점에서 스레드 풀을 이용하는것이 스레드를 직접 이용하는것보다 바람직하지만 주의사항이 있다.
        - k 스레드를 가진 스레드 풀은 k만큼의 스레드를 동시에 실행할 수 있다.
        - 이떄 잠을 자거나 I/O를 기다리거나 네트워크 연결을 기다리는 테스크가 있다면?
            - 이런 상황에서 스레드는 블록이된다.
                - 10개의 스레드를 갖는 스레드 풀에 20개의 테스크가 할당되고 5개의 스레드가 i/o를 기다리게 되면 결과적으로 나머지 5개가 다 처리해야한다.
                - 처음 제출한 테스크가 나중의 테스크의 제출을 기다리면 데드락이 발생할 수 있다.
            - 블록 상황에서는 테스크가 워커 스레드에 할당 된 상태를 유지하지만 아무 작업도 이루어지지 않는다.
            - `블록할 수 있는 테스크는 스레드 풀에 제출하지 말아야한다는 것이지만 항상 이를 생각할 수 없다.`
        - 프로그램 종료하기 전에 모든 스레드 풀을 종료하자. 자바는 이런 상황에서 `Thread.setDaemon` 메서드를 지원한다.


### 스레드의 다른 추상화 
- 테스크나 스레드가 메서드 호출 안에서 시작되면 그 메서드 호출은 반환하지 않고 작업이 끝나기를 기다리는 방식(스레드 생성 조인이 한번에 되는 포크/조인)
- 시작된 테스크를 내부 호출이 아니라 외부 호출에서 종료하도록 기다리는 조금 더 여유로운 포크/조인
    - 스레드 실행은 메서드를 호출한 다음의 코드와 동시에 실행되므로 데이터 경쟁 문제를 일으키지 않아야한다.
    - 기존 실행중이던 스레드가 종료되지 않은 상황에서 자바의 main() 메서드가 반환할 때
        - 어플리케이션 종료하지 못하고 모든 스레드가 실행을 끝낼 때까지 기다린다.
        - 어플리케이션 종료를 방해하는 스레드를 강제종료시키고 어플리케이션을 종료한다.

### 동기 API, 비동기 API
- 스트림을 활용하여 병렬하드웨어를 이용할 수 있다. 
    - 외부반복을 내부반복으로 바꿔야한다.
    - parellel 메서드를 이용하여 자바 런타임 라이브러리가 복잡한스레드 작업을 하지 않고 병렬로 요소 처리되도록 할 수 있다. 
    - 루프가 실행될 때 추축에 의존해야하는 프로그래머와는 달리 런타임 시스템은 사용할 수 있는 스레드를 더 잘고 있다는 것이 핵심

```java

    public static int  duringTime1(int hour) {
        return hour;
    }

    public static int duringTime2(int hour) {
        return hour;
    }


    public static void main (String[] args) throws InterruptedException {
        int hour = 30;

        Result result = new Result();

        Thread t1 = new Thread(() -> {
            result.left = duringTime1(hour);
        });

        Thread t2 = new Thread(() -> {
            result.right = duringTime2(hour);
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();
    }

    static class Result {
        private int left;
        private int right;
    }

```

- runnable이 대신 future api를 사용하면 더 단순화하지만 불필요한 코드가 있다.

```java
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        int hour = 30;

        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Future<Integer> submit1 = executorService.submit(() -> duringTime1(hour)); 
        Future<Integer> submit2 = executorService.submit(() -> duringTime2(hour));
        System.out.println(submit1.get() + submit2.get()); // submit은 자신의 task를 들고있는 Future를 반환
        executorService.shutdown();
    }
```

- 비동기로 해결 가능하다.
- 리액티브 형식 API
    - 람다를 기용한 계산은 합계를 정확히 알 수 없다.
    - 어떤 함수가 먼저 계산될지 모른다.
    - if-then-else를 사용하여 모든 콜백이 호출되었는지 확인한다.

```java

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        int hour = 30;
        Result result = new Result();

       f(hour, (int y) -> {
           result.left = y;
           System.out.println(result.left + result.right);
       });

        g(hour, (int y) -> {
            result.left = y;
            System.out.println(result.left + result.right);
        });
    }
    
    private static void f(int x, IntConsumer dealWithResult) {
        dealWithResult.accept(Person.f(x));
    }
    private static void g(int x, IntConsumer dealWithResult) {
        dealWithResult.accept(Person.g(x));
    }

```

### Sleep과 블록킹은 해롭다
- sleep 메서드를 호출하더라도 시스템은 자원을 점유하고 있다.스레드 풀에서 잠자는 task는 다른 task가 실행되지 못하게 막으므로 자원을 소비하게 된다.
- 스레드가 적을 때는 문제되진 않지만 많아지만 위험해진다.
- 두 코드가 모두 같은 동작을 수행하지만, 첫번째는 sleep 시간동안 스레드의 자원을 점유하지만, 밑의 스레드는 다른 작업이 실행할 수 있다. 

```java

    public static void main4(String[] args) throws InterruptedException, ExecutionException {
        work1();
        Thread.sleep(10000);
        work2();

        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        work1();
        scheduledExecutorService.schedule(ClassName::work2, 10, TimeUnit.SECONDS);
        scheduledExecutorService.shutdown();
    }

```

### 비동기 API에서는 예외
- Future를 구현한 CompletableFutrue에서는 런타임 get 메서드에 예외를 처리할 수 있는 기능을 제공, exceptionally와 같은 메서드도 제공된다.
- 리액티브 형식의 비동기 API에서는 return 대신 콜백이 호출되므로 에러시 호출할 콜백함수를 전달하면 된다.
- 콜백이 여러개 인경우 한 객체로 메서드를 감싸는 것이 좋다. Java 9 Flow API에서는 여러 콜백을 한 객체(네 개의 콜백을 감싸는 Subscriber)로 감싼다.

```java
void onComplete()
void onError(Throwable throwable)
void onNext(T item)
```

### CompletableFuture와 콤비네이터를 이용한 동시성
- CompletableFuturesms complete() 메서드를 이용하여 나중에 다른 스레드가 이를 완료하고, get()을 통해 값을 얻을 수 있다.
- f(x)의 실행이 끝나지 않는다면, get()을 기다려야하기 때문에 프로세스 자원을 낭비할 수 있다.

```java
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
        int x = 1337;

        CompletableFuture<Integer> objectCompletableFuture = new CompletableFuture<>();
        scheduledExecutorService.submit(() -> objectCompletableFuture.complete(f(x)));
        int b = objectCompletableFuture.get();
        
        scheduledExecutorService.shutdown();

```

- 이러한 문제점을 thenCombile을 활용한다.
- 두 개의 CompletableFuture 값을 받아 한 개의 새로운 값을 만든다. 처음 두작업이 끝나면 모두에 fn을 적용하고 블록되지 않은 상태로 Future를 반환한다.
- Future a와 Future b의 결과를 알지 못한 상태에서 thenCombine은 두 연산이 끝났을 때 스레드 풀에서 실행된 연산을 만든다.
- 결과를 추가하는 세번째 연산 c는 두 작업이 끝나기전까지 스레드에서 실행되지 않는다.

```java
CompletableFuture<V> thenCombine(CompletableFuture<U> other, BiFunction<T, U, V> fn);

ExecutorService executorService = Executors.newFixedThreadPool(10);
        int x = 1337;

        CompletableFuture<Integer> a = new CompletableFuture<>();
        CompletableFuture<Integer> b = new CompletableFuture<>();
        CompletableFuture<Integer> c = new CompletableFuture<>();

        executorService.submit(() -> a.complete(f(x)));
        executorService.submit(() -> b.complete(g(x)));
        System.out.println(c.get());

```

### 발행-구독 리액티브 프로그래밍
- Future와 CompletableFuture는 독립적 실행과 병렬성을 가지고 연산이 끝나면 get()으로 Function의 결과를 얻을 수 있다. -> Future는 한번만 실행해 결과를 제공
- 리액티브 프로그래밍은 시간이 흐르며 여려 Function 같은 객체를 통해 여러 결과를 제공한다.
- Java 9 에서는 java.util.concurrent.Flow 인터페이스에서 발행 - 구독 모델을 적용하여 리액티브 프로그래밍을 제공한다
    - 구독자가 구독할 수 있는 발행자
    - 이 연결을 구독
    - 이 연결을 통해 메시지 또는 이벤트를 전송한다.
- 두 플로우를 합치는 예제
    - 두 정보 소스로 부터 발행하는 이벤트를 합쳐 다른 구독자가 볼 수 있도록 발행하는 예를 통해 pub-sub의 특징을 간단히 확인할 수 있다.
    - C1 + C2라는 공식을 포함하는 C3를 만들자. 그러면 C1 또는 C2의 값이 바뀌면 C3에도 새로운 값이 반영된다.

```java

public class SimpleCell implements Flow.Publisher<Integer>, Flow.Subscriber<Integer> {

    private int value;
    private String name;
    private List<Flow.Subscriber> subscribers = new ArrayList<>();

    public SimpleCell(String name) {
        this.name = name;
    }

    public static void main(String[] args) {
        SimpleCell c1 = new SimpleCell("C1");
        SimpleCell c2 = new SimpleCell("C2");
        SimpleCell c3 = new SimpleCell("C3");

        c1.subscribe(c3);

        c1.onNext(10);
        c2.onNext(20);

        System.out.println("c1 : " + c1.value);
        System.out.println("c2 : " + c2.value);
        System.out.println("c3 : " + c3.value);
    }

    @Override
    public void subscribe(Flow.Subscriber<? super Integer> subscriber) {
        subscribers.add(subscriber);
    }

    @Override
    public void onSubscribe(Flow.Subscription subscription) {

    }

    @Override
    public void onNext(Integer item) {
        this.value = item;
        notifyAllSubscribers();
    }

    private void notifyAllSubscribers() {
        subscribers.forEach(subscriber -> subscriber.onNext(this.value));
    }


    @Override
    public void onError(Throwable throwable) {

    }

    @Override
    public void onComplete() {

    }
}

> Task :SimpleCell.main()
c1 : 10
c2 : 20
c3 : 10


```

- C1 + C2 구현하는 방법: 왼쪽 / 오른쪽 계산 결과를 저장할 수 있는 별도의 클래스가 필요.
    - 데이터가 발행자에서 구독자로 흐름에 착안해 개발자는 업스트림 또는 다운 스트림이라 칭한다.
    - 밑의 예제에서 새로운 값은 onNext() 메서드로 전달이 되고 notify 메서드를 통해 다운 스트림 onNext()호출로 전달된다.
    - 실제로 pub-sub 구조를 적용하려면, onError, onComplete와 같은 메서드를 통해 예외발생, 종료등을 알 수 있어야한다.
    - 만약 수많은 이벤트가 onNext로 전달된다면 어떻게 될것인가? 이것을 압력이라 부른다
    - 압력: 전달할 이벤트 또는 메시지를 제어할 수 있는 역압력 기법이 필요하다. 

```java
public class ArithmeticCell extends SimpleCell {

    private int left;
    private int right;

    public ArithmeticCell(String name) {
        super(name);
    }

    public void setLeft(int left) {
        this.left = left;
        onNext(left + this.right); // 셀 값을 갱신하고 구독자에게 알림
    }

    public void setRight(int right) {
        this.right = right;
        onNext(right + this.left);
    }

}



    public static void main(String[] args) {
        ArithmeticCell c3 = new ArithmeticCell("c3");
        SimpleCell c2 = new SimpleCell("c2");
        SimpleCell c1 = new SimpleCell("c1");

        c1.subscribe(c3::setLeft);
        c2.subscribe(c3::setRight);

        c1.onNext(10); // c3 : 10 
        c2.onNext(20); // c3 : 30
        c1.onNext(15); // c3 : 35
        
    }

```

- backpressure
    - 정보의 흐름속에서 backpressure로 제어 즉 subscriber-> publisher로 정보를 요청해야할 필요가 있다.
    - 예를 들어 subscriber가 적정의 데이터를 보내달라고 알려줄 필요가 있다.
    - `onSubscribe(Subscription subscription);`
    - 실제 backpressure
        - subscriber가 onSubscribe로 전달된 subscription 객체를 로컬에 저장
        - subscriber가 수많은 이벤트를 받지 않도록 onSubscribe, onNext, onError의 마지막 동작에 channel.request(1)을 추가하여 오직 한 이벤트만 요청한다.
        - 요청을 보낸 channel에만 onNext, onError 이벤트를 보낼도록 Publisher의 notify 코드를 수정
    - 장 단점
        - 여러 Subscriber가 있을 때 이벤트를 가장 느린속도로 보낼것인지, 각 Subscriber에게 보내지 않은 데이터를 저장할 별도의 큐를 구현할것인지
        - 구현한 큐가 너무 커진다면 고려해야한다.
        - Subscriber가 준비되지 않았다면 큐를 어떻게 고려할것인지.