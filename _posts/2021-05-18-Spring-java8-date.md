---
layout: post
title:  "JAVA 8 - DATE"
date:   2021-05-18 20:15
categories: spring, resolver
tags: [java, date]
---

### 자바 8 Date

#### 자바 8 날짜와 시간이 생긴이유
    - `java.util.Date` 클래스는 Mutable -> thread safe 하지 않음
    - 클래스 이름이 명확하지 않다. Date.getTime() 날짜에서 시간을 가져온다?
    - 월이 0부터 시작한다.

#### 주요 API
    - 기계용 시간과 인간용 시간으로 나뉜다.
        - 기계용 시간 : 1970년 1월 1일 0시 0분 0초 ~ 현재까지의 타임스탬프
        - 인간용 시간 : 연 월 일 시 분 초등을 표현
    - 타임스탬프는 Instance를 사용
    - 특정 날짜(LocalDate), 시간(LocalTime), 일시(LocalDateTime)를 사용할 수 있다.
    - 기간을 표현할 때는 Duration(시간 기반)과 Period(날짜 기반) 사용할 수 있다.
    - DateTimeFormatter를 사용해서 일시를 특정한 문자열로 포맷팅할 수 있다.

#### Date/Time Api
    - 기계적 시간 현재

```java
    Instant instant = Instant.now(); // 현재는 기준 시
    
```
    
    - 인간용 시간

```java
    LocalDateTime now = LocalDateTime.now(); // 현재시간  현 시스템 시간

```

    - Period

```java
    LocalDate localDate = LocalDate.now();
    LocalDate nextMonthDate = LocalDate.of(2021, Month.JULY, 18);
    Period period = Period.between(localDate, nextMothDate); 
    preriod.getDays(); // 30

    today.until(nextMonthDate).get(ChronoUnit.DAYS) // 30

```

    - Duration

```java
    Instant now = Instant.now();
    //now.plus(10, ChronoUnit.SECONDS);
    Duration between = Duration.between(now, now.plus(10, ChronoUnit.SECONDS)); // 10
```
    
    - LocatDateTime
    - 미리 정의되어 있는것을 쓰면 좋을것 같다 JAVA API

```java
    LocatDateTime now = LocatDateTime.now();
    DateTimeFormatter format = DateTimeFormatter.ofPattern("MM/dd/yyyy");
    now.format(format);

    LocalDate.parse("01/01/1994", format); // 1994-01-01

```


    