---
layout: post
title:  "[스프링] 타입 컨버터"
date:   2021-12-30 14:20:21 +0900
categories: java, spring
---

# 타입 컨버터

### request
- String data = request.getParameter("data");//문자 타입 조회
- data가 숫자인 경우 Integer.parseInt() 로 변경해야한다.
- 쿼리스트링으로 들어오는 것은 다 문자열이다.
- @RequestParam을 사용하면 중간에서 타입을 변환해줘서 따로 처리를 안해줘도 된다.


### 스프링과 타입 변환
- 컨버터 인터페이스

```java

package org.springframework.core.convert.converter;

public interface Converter<S, T> {
    T convert(S source); // S를 받아 T로 반환
}

```

- 스프링에서 추가적인 타입 변환이 필요하면 이 인터페이스로 구현 등록하면 된다.
- 스트링에서 정수형으로 바꾸는 컨버터

```java

public class StringToIntegerConverter implements Converter<String, Integer> {

    @Override
    public Integer convert(String source) {
        log.info("convert source={}", source);
        return Integer.valueOf(source);
    }
}

```

### 컨버전 서비스
- 타입 컨버터를 하나하나 직접 찾아서 변환하는것은 불편한대 이것을 해결해주는 것이 `컨버전 서비스`이다.
- 스프링은 개별 컨버터를 모아두고 그것들을 묶어 편리하게 사용할 수 있는 기능을 제공
- 컨버팅이 가능한가, 확인하는 기능, 컨버팅 기능을 제공
- 컨버터를 등록할 때는 같은 타입 컨버터를 명확하게 알아야하지만, 컨버터를 사용하는 입장에서는 어떤 타입 컨버터인지 몰라도 된다. => 등록과 사용이 분리되어 있다.
- 타입 컨버터들은 모두 컨버전 서비스에 캡슐화 되어 제공된다. => 사용자는 컨버전 서비스에만 의존하면 된다.
- 스프링 빈으로 등록함으로써 따로 컨버터를 등록하지 않아도 컨버팅이 이루어진다.

```java

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
      
        registry.addConverter(new StringToInregerConverter());
        
    }
}

```

- 처리과정 : @RequestParam을 처리하는 ArgumentResolver인 RequestParamMethodArgumentResolver에서 컨버전서비스를 사용하여 타입을 변경



