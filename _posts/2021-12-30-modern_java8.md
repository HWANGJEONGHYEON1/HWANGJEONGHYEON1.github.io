---
layout: post
title:  "Modern Java - 8"
date:   2021-12-30 11:20:21 +0900
categories: java
---

> 기본에 충실하자 - 7

# CompletableFuture - 안정적 비동기 프로그래밍

### Future의 단순한 활용
- java5 부터 미래의 어느 시점에 결과를 얻는 모델에 활용할 수 있도록 Future 인터페이스를 제공할 수 있다.
- 시간이 걸리는 작업들을 Future 내부에 설정하여 호출자 스레드가 결과를 기다리는 동안 다른 유용한 작업을 할 수 있다.
- 시간이 오래걸리는 작업을 다른 스레드에서 처리하고 메인 스레드에서는 다른 작업들을 미리 수행할 수 있다.
- executorService에서 제공하는 스레드가 오래걸리는 작어블 처리하는 동안 다른 스레드로 다른 작업을 동시에 실행할 수 있다.
- 작업의 결과가 필요한 시점이 되면 get() 메서드로 호출할 수 있다. 그때까지 일이 마쳐진 상태이면 바로 반환을 하고 아니면 스레드를 블록한다.
- 오래 걸리는 작업이 영원히 끝나지 않으면 영원히 블록상태가 된다. 그래서 스레드가 대기할 timeout을 정해놔야한다.

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<Double> future = executorService.submit(new Callable<Double>() {
            @Override
            public Double call() throws Exception {
                return doSomethingEvent(); // 시간이 걸리는 작업
            }

            private Double doSomethingEvent() {
                System.out.println("## future");
                return 0d;
            }
        });

        anotherEvent();
        future.get(1, TimeUnit.SECONDS);
    }

    private static void anotherEvent() {
        System.out.println("다른 작업");
    }

    // 출력
    // 다른 작업
    // ## future
```

### Future의 한계
- Future의 인터페이스가 비동기 계산이 끝났는지 확인할수 있는 isDone, 계산이 끝나길 기다리는 메서드, 결과 회수 메서드등을 제공하지만 이것들만으로는 동시 실행 코드를 구현하기 어렵다.
- 오래걸리는 A 계산이 끝난다면, B를 실행할 수 있는 로직이 필요하다.
- 다음과 같은 선언형 기능이 있으면 유용할 것이다.
    - 두개의 비동기 계산을 하나로 합친다. 두 계산결과가 서로 독립적일 수도, 두번째가 첫번째에 영향을 미칠수도 있다.
    - Future 집합이 실행하는 모든 task 완료를 기다린다.
    - Future 집합에서 가장 빠른 task를 기다렸다가 결과를 얻는다.
    - Future를 수동으로 완료시킨다.
    - Future 완료 동작에 반응한다(결과를 기다리며 블록되지 않고 결과가 준비되었다는 알림을 받은 다음 Future의 결과로 원하는 추가동작을 수행가능)
- 선언형으로 Future를 사용할 수 있는 CompletableFuture 클래스에 대해 알아본다.

### CompletableFuture 비동기 어플리케이션
1. 고객에게 비동기 API를 제공하는 방법을 배운다.
2. 동기 API를 사용해야할 떄 코드를 비블록으로 만드는 방법을 배운다. 두 개의 동작을 파이프라인으로 만드는 법과 두 개의 동작결과를 하나의 비동기계산으로 합치는 방법
3. 비동기 동작의 완료에 대응하는 방법을 배운다.

```java

/**
* API 호출되는 경우 비동기 동작이 완료될 때까지 블록 상태가 된다.
*/ 
public class Shop {

    public double getPrice(String product) {
        return calculatePrice(product);
    }

    private double calculatePrice(String product) {
        delay();
        return new Random().nextDouble() * product.charAt(0) + product.charAt(1);
    }

    public void delay() {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 비동기 API를 활용함으로써 즉시 Future를 반환한다. 요청자는 전달받은 Future를 이용하여 적절한 시점에 결과를 얻을 수 있고, 그 동안 다른 작업이 가능하다.
    // 그동안 에러가 발생했다면? 다른 스레드를 만들어 계산을 하므로, 해당 스레드에만 영향을 주고 이후에 진행되는 다른 shop api를 가져오는 과정은 진행된다.
    // 요청자는 결과적으로 get 메서드가 반환될 때까지 영원히 기다리게 될 수 있다. => get 메서드를 오버라이딩하여 timeout을 설정한다.
    public Future<Double> getPriceAsync(String product) {

        return CompletableFuture.supplyAsync(() -> calculatePrice(product));
    }
}

```

- 병렬 스트림의 버전의 코드는 지정한 숫자의 상점에 하나의 스레드를 할당하여 네 개의 작업을 병렬로처리, 상점이 하나 추가되는 경우(스레드 개수가 4개일 때)?
- 순차 실행인 경우는 5초 이상, 나머지는 2초이상 소모 될것이다.
- 병렬 스트림과 비동기는 비슷한 결과를 보이지만 비동기는 병렬 스트림에 비해 작업에 사용할 다양한 Excutor를 지정할 수 있다.

```java

    public List<String> findPrices(String product) {
        return shops.stream()
                .map(shop -> String.format("%s is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }

    public List<String> findPricesParallel(String product) { // 병렬
        return shops.parallelStream()
                .map(shop -> String.format("%s is %.2f", shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }

    public List<String> findPricesCompletable(String product) {
        List<CompletableFuture<String>> priceFutures = shops.stream()
                .map(shop -> CompletableFuture.supplyAsync(() -> String.format("%s is %.2f", shop.getName(), shop.getPrice(product))))
                .collect(Collectors.toList());

        return priceFutures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());
    }

```

### 커스텀 익스큐터 만들기
- 스레드가 많을수록 사용하지 않는 스레드가 많아지고 서버가 크래시 날 수 있으므로 적정한 개수를 지정하는 것이 중요하다
- 스레드 풀이 너무 크면 CPU 와 메모리가 자원을 경쟁하느라 시간이 낭비된다. 반면 적으면 CPU 코어를 활용하기 어렵다
- 브라이언 게츠는 적정 스레드 수 공식 => (이용가능한 프로세서가 반환하는 값) CPU 활용 비율 (0-1) (1 + 대기시간 / 계산시간)

```java

private final Executor executor = Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory() {
    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setDaemon(true); // 프로그램 종료를 방해하지 않는 데몬 스레드를 사용.
        return t;
    }
});
```

### 비동기 작업 파이프라인 만들기
- 상점이 하나의 할인 서비스를 사용한다고 가정
- 가져온 가격을 파싱해서 객체로 생성
- 해당 객체를 사용해서 discount 서비스에서 할인율을 가져오고, 할인을 적용

```java
// 간단한 작업
public List<String> findPrices(String product) {
   return shops.stream()
   .map(shop -> shop.getPrice(product))
   .map(Quote::parse)
   .map(Discount::applyDiscount)
   .collect(toList());
}

// 비동기 작업
public Stream<CompletableFuture<String>> findPricesStream(String product) {
    return shops.stream()
        .map(shop -> CompletableFuture.supplyAsync(() -> shop.getPrice(product), executor))
        .map(future -> future.thenApply(Quote::parse))
        .map(future -> future.thenCompose(quote -> CompletableFuture.supplyAsync(() -> Discount.applyDiscount(quote), executor)));
  }
```

- supplyAsync : 비동기적으로 상점에서 정보 조회 -> Stream<CompletableFuture<String>> 리턴
- Quote 파싱 : 객체 생성 -> thenApply를 이용하여 실행
    - thenApply는 CompletableFuture가 끝날 때까지 블록하지 않는다. CompletableFuture가 동작을 완전히 완료한다음 thenApply 메서드로 전달된 람다표현식을 적용할 수 있다.
- thenCompose : 첫 번째 연산결과를 두 번째 연산으로 전달. CompletableFuture에 thenCompose를 홀출하고, Function을 넘겨주는 식으로 두 CompletableFuture를 조합한다.
- Function은 첫번째 CompletableFuture 반환 결과를 인수로 받고 두 번째 CompletableFuture를  반환하는대 CompletableFuture는 첫번째 CompletableFuture의 결과를 입력으로 사용

### 독립 CompletableFuture와 비 독립 CompletableFuture 합치기
- 실 생활에서는 위처럼 한쪽이 의존관계가 아니라, 독립적으로 외부 API를 두 개 이상을 호출하여 가져와야할 때가 있다. 그런 경우 thenCombine을 사용

```java
public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(null, other, fn);
    }
 shop.getPriceAsync("mocha").thenCombine(shop.getPriceAsync("americano"), Integer::sum).join();


//completeOnTimeOut(반환값 : CompletableFuture)은 특정 시간안에 응답이 오지 않는 경우 기본 값을 전달할 수 있다.
//무한정 대기하는것을 막기 위해 TimeOut을 설정
Future<Double> futurePriceInUSD = 
  CompletableFuture.supplyAsync(() -> shop.getPrice(product)) // 제품가격을 요청하는 첫번째 테스크 생성.
                   .thenCombie(CompletableFuture.supplyAsync(() -> exchangeService.getRate(Money.EUR, Money.USD)), (price, rate) -> price * rate) // USD, EUR의 환율 정보를 요청하는 독립적인 두 번째 테스크 생성.
                   .orTimeOut(3, TimeUnit.SECONDS); // 3초 뒤 작업이 완료되지 않으면 TimeoutException을 발생시키도록 설정



```
