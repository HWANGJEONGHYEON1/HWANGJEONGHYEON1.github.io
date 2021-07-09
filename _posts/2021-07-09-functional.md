---
layout: post
title:  "functional interface"
date:   2021-07-08 22:10
categories: functional, interface
tags: [functional, interface]
---

## Functional Interface

### Predicate<T>
- Predicate<T> boolean test(T t) 매개변수 T, 반환 boolean

```java
public class Predicate {

    private static List<Integer> numList = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

    public static int count(IntPredicate predicate){
        int count = 0;
        for(int num : numList){
            if(predicate.test(num)){
                count ++;
            }
        }
        return count;
    }

    public static void main(String[] args) {
        int evenNumCount = count(num -> {
            if (num % 2 ==0) {
                return true;
            }

            return false;
        });

        System.out.println("짝수 갯수 : " + evenNumCount);

        int oddNumCount = count(num -> {
            if (num % 2 == 0) {
                return true;
            }

            return false;
        });

        System.out.println("홀수 개수 : " + oddNumCount);
    }
}
```

### Consumer
- Consumer<T> void accept(T t) 매개변수 T, 반환 없음
- Consumer 함수적 인터페이스는 리턴값이 없는 accept() 메소드를 가지고 있다.
- Consumer는 단지 매개값을 소비하는 역할만 하며, 소비한다는 말은 사용만하고 리턴값이 없다는 뜻이다.


```java
import java.util.function.BiConsumer;
import java.util.function.Consumer;
import java.util.function.DoubleConsumer;
import java.util.function.ObjIntConsumer;

public class ConsumerExample {

    public static void main(String[] args) {
        Consumer<String> consumer = s -> System.out.println(s + " world!");
        consumer.accept("hello");

        BiConsumer<String, String> biConsumer = (t, u) -> System.out.println(t + " " + u);
        biConsumer.accept("hello", " world !");

        DoubleConsumer doubleConsumer = d -> System.out.println("num : " + d);
        doubleConsumer.accept(10);

        ObjIntConsumer<String> objIntConsumer = (t, u) -> System.out.println(t + u);
        objIntConsumer.accept("num : ", 10);
    }

}

```

### Supplier
- Supplier 함수형 인터페이스는 매개값은 없고 리턴값이 있는 getXXX() 메소드를 가지고 있다.
- T get() : T를 리턴

```java
import java.util.function.IntSupplier;

public class SupplierExample {

    public static void main(String[] args) {
        IntSupplier intSupplier = () -> {
            int num = (int) (Math.random() * 6 + 1);
            return num;
        };

        int num = intSupplier.getAsInt();
        System.out.println(num);
    }

}
```

### Function
- 매개값과 리턴값이 있는 applyXXX() 메서드를 가지고 있다.
- 메서드는 매개값을 리턴값으로 매핑하는 역할을 한다.
- R apply(T t) : 객체 T를 R로 매핑

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.function.ToDoubleFunction;
import java.util.function.ToIntFunction;

public class FunctionExample {
    private static List<Student> list = Arrays.asList(new Student("a", 100, 100), new Student("b", 30,30));

    public static void printString(Function<Student, String> function) {
        for (Student student : list) {
            System.out.println(function.apply(student) + " ");
        }
        System.out.println(" ");
    }

    public static void printInt(ToIntFunction<Student> function) {
        for (Student student : list) {
            System.out.println(function.applyAsInt(student) + " ");
        }

        System.out.println();
    }

    public static double getAverage(ToDoubleFunction<Student> function) {
        int sum = 0;
        for (Student student : list) {
            sum += function.applyAsDouble(student);
        }

        double avg = sum / list.size();

        return avg;
    }

    public static void main(String[] args) {

        System.out.println("학생이름");
        printString(t -> t.getName());

        System.out.println("영어점수");
        printInt(t -> t.getEnglishScore());

        System.out.println("수학점수");
        printInt(e -> e.getMathScore());

        double englishAvg = getAverage(t -> t.getEnglishScore());
        double mathAvg = getAverage(t -> t.getMathScore());
        System.out.println("영어점수 평균 " + englishAvg);
        System.out.println("수학점수 평균 " + mathAvg);


    }
}

class Student {
    private String name;
    private int mathScore;
    private int englishScore;
    public Student(String name, int mathScore, int englishScore) {
        this.name = name;
        this.mathScore = mathScore;
        this.englishScore = englishScore;

    }
    public String getName() {
        return name;
    }
    public int getMathScore() {
        return mathScore;
    }
    public int getEnglishScore() {
        return englishScore;
    }
}
```

### Operator
- Function과 동일하게 매게 변수와 리턴 값이 있는 applyXXX() 메소드를 가지고 있다. 매개값을 리턴값으로 매핑하는 역할보다 매개값을 이용해서 연산ㅇ르 수행한 후 동일한 타입으로 리턴 값 제공

```java
import java.util.function.IntBinaryOperator;

public class OperatorExample {

    private static int[] arg = {15, 20, 10 ,50};

    public static int maxOrMin(IntBinaryOperator operator) {
        int result = arg[0];

        for (int age : arg) {
            result = operator.applyAsInt(result, age);
        }

        return result;
    }

    public static void main(String[] args) {
        int max = maxOrMin((a, b) -> {
            if (a>=b) {
                return a;
            }
            return b;
        });

        int min = maxOrMin((a, b) -> {
            if (a <= b) {
                return a;
            }
            return b;
        });

        System.out.println("최대값 : " + max);
        System.out.println("최소값 : " + min);
    }
}

```