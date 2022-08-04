---
layout: post
title:  "Effectivce Java - 7"
date:   2022-06-23 16:20:21 +0900
categories: java
---

> #2 기본에 충실하자 

# 일반적인 프로그래밍 원칙

## item49) 지역변수의 범위를 최소하하라
- 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 '가장 처음 쓰일 때 선언하기'다.
- 거의 모든 지역변수는 선언과 동시에 초기화해야한다.
- 지역변수 범위를 최소하하는 마지막 방법은 메서드를 작게 유지하고 한 가지 가닝으 집중하는 것이다.


## item50) 전통적인 for 보다는 for-each문을 사용하라
- for-each를 쓸 수 없는 세가지
    - 파괴적인 필터링
        - 컬렉션을 순회하면서 선택된 원소를 제거해야한다면 반복자의 remove를 호출해야한다.
        - 자바8부터는 Collcetions의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는일을 피할 수 있다.
    - 변형
        - 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야한다면 리스트의 반복자나 배열의 인덱스를 사용해야한다.
    - 병렬 반복
        - 여러 컬렉션을 병렬로 순회해야 한다면 각자 의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야한다.
- 전통적인 for 문과 비교했을 때, 명료하고, 유연하고, 버그를 예방해준다. 성능저하도 없다.


## item51) 라이브러리를 익히고 사용하라
- 무작위 정수하나를 생성하고 싶다고 해보자.

```java
// 흔하지만 문제가 심각한 코드
static Random rnd = new Random();

static int random(int n) {
    return Math.abs(rnd.nextInt()) % n;
}
```
- 문제
    1. n이 그리 크지 않은 2의 젭곱수라면 얼마지나지않아 같은 수열이 반복
    2. 2의 제곱수가 아니라면 몇몇 숫자가 평균적으로 더 자주반환한다.
- 표준 라이브러리를 사용하면 그 코드를 작성한 전문가의 지식과 앞서 사용한 다른 프로그래머들의 경험을 활용할 수 있다.
- 자바 7부턴 Random을 사용하지 않고, ThreadLocalRandom으로 대체하면 잘 작동한다.
- 두번째 이점은 핵심적인 일과 크게 관련없는 문제를 해결하느라 시간을 낭비하지 않아도된다.
- 세번쨰 이점은 따로 노력하지 않아도 성능이 지속적으로 개선이된다.
- 네번째 이점은 기능이 점점많아진다. 라이브러리에 부족한 부분이 있다면 개발자 커뉴니티에서 이야기가 나오고 다음 릴리즈에 해당 기능이 추가될 수 있다.
- 자바 개발자라면 util, lang, io 에 익숙해져야한다.

## item60) 정확한 답이 필요하다면 float와 double은 피하라
- float과 double은 공학 계산용으로 설계되었다.
    - 이진 부동소수점 연산으로 쓰이며, `근사치`로 계산하도록 설계
- `특히 금융 관련 계산과는 맞지 않는다.`
- System.out.print(1.03 - 0.42);
    - 0.6100000000001
- 코딩시의 불편함이나 성능저하를 신경쓰지 않는다면 BigDecimal을 사용하라
- 성능이 중요하고 소수점을 직접 추적할 수 있고 숫자가 크지 않다면 int, long을 사용하라.
- 여덟자리 십진수를 long, 넘어간다면 BigDecimal을 사용해야한다.

## item61) 박싱된 기본타입 보다는 기본타입을 사용하라
- int, double, boolean에 대응하는 박싱된 기본타입은 Integer, Double, Boolean 이다.
- 기본타입과 박싱된 기본타입의 주된 차이 3가지
    1. 기본타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별셩이란 속성을 갖는다.
    2. 기본 타입은 언제나 유요하나, 박싱된 기본타입은 유효하지 않은 값, 즉 NULL을 가질 수 있다.
    3. 메모리 사용면에서 더 효율적이다.

```java

        Comparator<Integer> naturealOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);


        Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j) ? 0 : 1;
        int compare = naturalOrder.compare(new Integer(42), new Integer(42));
        System.out.println(compare); // 결과는 0이어하지만, 1이 나온다. ?

```

- 위 방식이 잘못된 이유는 i 와 j가 참조하는 오토박싱된 Integer 인스턴스는 기본타입값으로 변환한다.
- 두번째 검사에서 i == j 는 두 객체 참조 식별성을 검사한다. i와 j가 서로 다른 Integer의 인스턴스라면 같지 않게 된다.
- 박싱된 기본타입에 연산자 == 를 사용하면 오류가 일어난다.
- 밑의 방식으로 해결하자

```java
    Comparator<Integer> naturealOrder = (iBox, jBokx) -> {
        int i = iBox;
        int j = jBox;
        return (i < j) ? -1 : (i == j) ? 0 : 1;
    }
```

- 오토박싱이 박싱된 기본타입을 사용할 때 번거러움을 줄여주지만, 위험까지는 없애주지 않는다.
- 언박싱 과정에서 NPE를 발생시킬 수 있다.


## item62) 다른 타입이 적절하다면 문자열 사용을 피하라
- 문자열은 열거 타입을 대신하기에 적합하지 않다.
- 문자열은 혼합 타입을 대신하기에 적합하지 않다.
    - `String compuondKey = className + "#" + i.next(); `
    - 각 요소를 개별로 접근하려면 문자열을 파싱해야해서 느리고, 오류 가능성도 커진다.
- 문자열은 권한을 표현하기에 적합하지 않다.


## item63) 문자열 연결은 느리니 주의하라
- 문자열 열결 연선자로 문자열을 n개를 잇는 시간은 n 제곱의 비례한다.
```java
String result = "";
for (int i = 0; i < n; i++) {
    result += "a";
}
```
- 문자열을 연결할 경우 양쪽의 내용 모두 복사해야하므로 성능 저하는 피할 수 없다.
- 성능을 포기하고 싶지 않다면 StringBuilder를 사용하자

## item64) 객체는 인터페이스를 사용해 참조하라
- 적합한 인터페이스가 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스로 선언하라

```java

// 좋은 예
Set<Son> set = new LinkedHashSet<>();

// 나쁜 예
LinkedHashSet<Son> set = new LinkedHashSet<>();
```
- 적합한 인터페이스가 없다면 클래스로 참조해야한다.
    - 값 클래스 (Integer, String)
- 적합한 인터페이스가 없다면 클래스의 계층구조 중 필요한 기능을 만족하는 가장 덜 구체적인 클래스를 타입으로 사용하자

## item65) 리플렉션보다는 인터페이스를 사용하라
- 리플렉션을 사용하면 임의의 클래스에 접근할 수 있다.
    - Class 객체가 주어지면 생성자, 메서드, 필드에 해당하는 인스턴스를 가져올 수 있고, 인스턴들로는 그 클래스의 멤버이름, 필드타입, 메서드 시그너처등을 가져올 수 있다.
- 나아가 인스턴스를 이용하여 연결된 실제 생성자, 메서드, 필드를 조작할 수 있다.
- 단점
    - 컴파일타임 타입 검사가 주는 이점을 하나도 누릴 수 없다.
    - 리플렉션을 이용하면 코드가 지저분해지고 장황해진다.
    - 성능이 저하된다.
- 코드 분석도구나 의존관계 주입 프레임워크처럼 리플렉션을 써야하는 복잡한 애플리케이션이 몇 가지 있다.
    - 리플렉션을 줄여가고 있음
- 단점을 취하고 이점만 취할 수 있는 방법
    - 리플렉션은 인스턴스 생성만 사용하고 만든 인스턴스는 인터페이스나 상위 클래스를 참조해 사용하자

```java
main() {
    Class <? extends Set<String>> cl = null;

    try {
        cl = (Class <? extends Set<String>> cl) Class.forName(args[0]);
    } catch(ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }

    Constructor<? extedns Set<String>> cons = null;
    try {
        cons = cl.getDeclearedConstructor();
    } catch(ClassNotFoundException e) {
        fatalError("클래스를 찾을 수 없습니다.");
    }

    Set<String> s = null;
    try {
        s = cons.newInstance();
    } catch (IllegalAccessException e) {
        fatalError("생성자에 접근할 수 없습니다.");
    } catch (Instantication e) {
        fatalError("클래스를 인스턴스화 할 수 없습니다.");
    } catch (InvocationTargetException e) {
        fatalError("생성자가 예외를 던졋습니다.");
    } catch (ClassCastException e) {
        fatalError("Set을 구현하지 않은 클래스입니다..");
    }

    s.addAll(Arrays.asList(args).subList(1, args.length));
    sout(s);
}
```

- 위 예제는 단점 두가지가 있다.
    - 런타임에 총 여섯가지나 되는 예외를 던질 수 있다.
        - 인스턴스를 리플렉션없이 생성했다면, 컴파일타임에 잡아낼 수 있는 예외이다.
    - 클래스 이름만으로 인스턴스를 생ㅅ어해내기 위해 25줄이나 되는 코드를 작성했다.
        - 리플렉션이 아니라면 생성자호출 한줄로 끝냈다.
- 리플렉션 예외를 각각 잡는 대신 리플렉션 예외 상위 클래스인 ReflectiveOperationException(자바 7이상)이 지원된다.

## item66) 네이티브 메서드는 신중히 사용하라

