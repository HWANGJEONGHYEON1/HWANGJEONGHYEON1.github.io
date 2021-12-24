---
layout: post
title:  "Modern Java - 5"
date:   2021-12-24 16:20:21 +0900
categories: java
---

> 기본에 충실하자 - 5

# 새로운 날짜와 시간 API
- 자바 8 이전에는 결함이존재.
- 새로운 날짜 클래스는 불변클래스
- 스레드 세이프

### LocalDate, LocalDateTime, Instant, Duration, Period
- LocalDate, LocalTime
    - LocalDate 인스턴스는 시간을 제외한 날짜를 표현하는 불변 객체
    - 어떤 시간대 정보도 포함하지 않는다.
    - 정적 팩토리 메소드 of를 활용하여 LocalDate 인스턴스를 만들 수 있다.
    - 팩토리 메서드 now()는 시스템 시계의 정보를 이용해 현재 날짜정보를 얻는다.

```java
LocalDate localDate = LocalDate.of(2021, 12, 24);
localDate.getYear();
localDate.getMonth();

// 시간대
LocalTime time = LocalTime.of(16, 00, 00) // 16:00:00
time.getHour()

// 날짜와 시간 문자열로 인스턴스 생성 방법
LocalDate date = LocalDate.parse("2021-12-24");
LocalTime time = LocalTime.parse("16:00:00"); // DateTimeFormatter 파라미터 전달

// 날짜 & 시간 조합
LocalDateTime dateTime = LocalDateTime.of(2021, Month.DECEMBER, 24, 16, 16, 16);

```

- Instant
    - 유닉스 에포크 시간을 기준으로 특정 지점까지의 시간을 초로 표현
    - 팩토리 메서드 ofEpochSecond에 초를 넘겨줘서 Instant 클래스 인스턴스를 만들 수 있다.
    - Instant 클래스는 나노초의 정밀도를 제공

- Duration, Period
    - Temporal 인터페이스는 특정 시간을 모델링하는 객체의 값을 어떻게 읽고 조작할지 정의한다.
    - Duration은 between으로 두시간 객체 사이의 지속시간을 만들 수 있다.
    - LocalDateTime은 사람이 사용하도록, Instant는 기계가 사용하도록 만들어졌다.
    - Period 클래스의 팩토리 메서드 between을 이용하면 두 LocalDate의 차이를 확인할 수 있다. 
    - 다양한 팩토리 메서드 제공
    - `불변성` : 스레드 세이프하며 도메인 모델의 일관성을 유지하는데 좋은 특성

```java
Duration d1 = Duration.between(time1, time2); 
Duration d1 = Duration.between(dateTime1, dateTime2);
Duration d1 = Duration.between(instant1, instant2);

Preiod diffDay = Preiod.between(LocalDate.of(2021,12,24), LocalDate.of(2021,12,30)) // 6

Duration fiveMin = Duration.ofMinutes(5);
Duration fiveMin = Duration.of(5, ChronoUnit.MINUTES);

Period tenDays = Preiod.ofDays(10);
Period twoWeeks = Preiod.ofWeeks(2);
```

### 날짜 변경
- 모든 메서드는 불변성을 가진다. (기존 객체를 변경하지 않음)

```java
LocalDate date = LocalDate.of(2021, 12, 24);
date.withYear(2022) // 2022-12-24
date.withDayOfMonth(25) // 2021-12-24
date.with(ChronoField.MONTH_OF_YEAR, 2) // 2021-2-24

date.plusWeek(1) // 2021-12-31
date.minuYear(1) // 2020-12-24
date.plus(6, ChronoUnit.MONTHS) // 2021-06-24

```

### TemporalAdjusters 의 팩토리 메소드 

```
dayOfWeekInMonth 서수 요일에 해당하는 날짜를 반환
firstDayOfMonth 현재 달의 첫 번쨰 날짜를 반환
firstDayOfNextMonth 다음 날의 첫 번째 날짜를 반환
firstDayOfNextYear 내년의 첫 번째 날짜를 반환
firstDayOfYear 올해의 첫 번째 날짜를 반환
firstInMonth 현재 달의 첫 번째 요일에 해당하는 날짜를 반환
lastDayOfMonth 현재달의 마지막 날짜를 반환
lastDayOfNextMonth 다음 달의 마지막 날짜를 반환
lastDayOfNextYear 내년의 마지막 날짜를 반환
lastDayOfYear 올해의 마지막 날짜를 반환
lastInMonth 현재 달의 마지막 요일에 해당하는 날짜 반환
next / previous 현재 달에서 현재 날짜 이후로 지정한 요일이 처음으로 나타나는 날짜를 반환
nextOrSame / previousOrSame 현재 날짜 이후로 지정한 요일이 처음 / 이전으로 나타나는 날짜를 반환
```


### 날짜 시간 객체 출력 및 파싱

- DateTimeFormmatter 클래스는 BASIC_ISO_DATE와 ISO_LOCAL_DATE등의 상수를 미리 정의하고 있다.
- DateTimeFormmatter를 활용하여 날짜나 시간을 특정 형식의 문자열로 만들 수 있다.

```java
LocalDate date = LocalDate.of(2021, 12, 24);
date.format(DateTimeFormmatter.BASIC_ISO_DATE);// 20211224
date.format(DateTimeFormmatter.ISO_LOCAL_DATE);// 2021-12-24

// 날짜 객체로 변경
LocalDate.parse("20211224", DateTimeFormatter.BASIC_ISO_DATE);
LocalDate.parse("2021-12-24", DateTimeFormatter.ISO_LOCAL_DATE);

// 정적 팩토리 메소드 제공
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate localDate = LocalDate.of(2014, 3, 18);
String formattedDate = localDate.format(formatter);
```

### ZoneId 클래스 등장
- 시간대를 간단하게 처리
- getRules()를 통하여 시간대의 규정을 정할 수 있다.

```java
ZoneId romeZone = ZoneId.of("Europe/Italy");

// zoneId는 지역/도시 형식으로 이루어지며 IANA TIME ZONE DB에서 제공하는 지역 집합 정보를 사용
ZoneId zoneId = TimeZone.getDefault().toZoneId();
```

