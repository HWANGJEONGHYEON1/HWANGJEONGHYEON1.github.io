---
layout: post
title:  "[study] kafka - 4"
date:   2022-10-14 18:20:21 +0900
categories: kafka
---

> 카프카 스터디 5

## 카프카 스트림즈
- 토픽에 적재된 데이터를 실시간으로 변환하여 다른 토픽에 적재하는 라이브러리
- 카프카에서 공식적으로 지원하는 라이브러리
- 카프카 클러스터를 운영하면서 실시간 스트림 처리를 해야하는 필요성이 있다면, 카프카 스트림즈 어플리케이션을 개발하는 것이 좋다.

### 스트림즈를 사용해야하는 이유
- 다양한 기능을 스트림즈DSL로 제공하여 기능을 확장할 수 있다.
- 컨슈머와 프로듀서를 조합하여 스트림즈가 제공하는 기능과 유사하게 만들 수 있다.
    - 하지만 장애 허용 시스템등의 특징들은 컨슈머와 프로듀서만으로 조합하기 완벽하게 구현하기 힘들다.

### 스트림즈 내부구조
- 스트림즈 어플리케이션은 내부적으로 스레드를 1개 이상 생성할 수 있으며, 스레드는 1개 이상의 테스크를 가진다.
- 스트림즈의 `테스크`는 스트림즈 어플리케이션을 실행하면 데이터 처리 최소 단위이다.

### 토폴로지
- 2개 이상의 노드들과 선으로 이루어진 집합
- 카프카 스트림즈는 트리형 토폴로지
- 토폴로지를 이루는 노드 하나를 프로세서, 노드와 노드를 이은 선은 스트림이라고 부른다.
    - 소스프로세서: 데이터를 처리하기 위해 최초로 선언(데이터를 가져오는 역할)
    - 스트림 프로세스: 데이터를 변환 하는 역할
    - 싱크 프로세서: 카프카 토픽으로 저장하는 역할, 스트림즈로 처리된 데이터의 최종 종착지 

## 스트림즈 DSL, 프로세서 API
- 스트림즈 DSL
    - 스트림 프로세싱에서 쓰일만한 다양한 기능들을 자체 API로 변환 로직을 어렵지 않게 개발할 수 있다.
    - 메시지 값을 기반으로 토픽 분기처리
    - 지난 10분간 들어온 데이터의 개수 집계
    - 
- 프로세서 API 
    - 스트림즈 DSL로 구현되지 않는 것들을 제공
    - 메시지 값의 종류에 따라 토픽을 가변적으로 전송
    - 일정한 시간 간격으로 데이터 처리


### 스트림즈 DSL
- 레코드의 흐름을 추상화한 3가지 개념
    - KStream
    - KTable
    - GlobalKTable

### KStream
- 레코드의 흐름을 표현한 것으로 메시지키와 값으로 구성되어있다.
- 컨슈머로 토픽을 구독하는 것과 동일한 선상에서 사용하는 것이라고 볼 수 있다.

### KTable
- 메시지 키를 기준으로 묶어서 사용
- KStream 토픽의 모든 레코드를 조회할 수 있지만, KTable은 유니크한 메시지 키를 기준으로 가장 최신의 레코드를 사용한다.
- 데이터를 조회하면 메시지 키를 기준으로 가장 최신에 추가된 레코드의 데이터가 출력된다.
- 새로 데이터를 적재할 때 동일한 메시지 키가 있을 경우 데이터가 업데이트 되었다고 볼 수 있다.
    - 메시지 키의 가장 최신 레코드가 추가 되었기 때문에
- 예시 데이터, 사용자의 주소

### 코파티셔닝
- 조인을 하는 2개의 데이터의 파티션 개수가 동일하고, 파티셔닝 전략을 동일하게 맞추는 작업.
- kStream과 KTable을 조인하려면 반드시 con-partitioning을 조회해야한다.
- 파티션 개수가 다르다면 코파티셔닝 에러가 발생한다.

### GlobalKTable
- 코파티셔닝이 되지 않은 KStream과 KTable을 조인해서 사용하고 싶다면, KTable을 GlobalKTable로 선언하여 사용하면 된다.
- 큰 용량의 테이블은 사용하지 않는것이 좋다. 모든 파티션에 대해서 데이터를 다룬다.

## 스트림즈 DSL 중요 옵션
- bootstrap.servers: 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트이름:포트를 1개 이상 작성한다. 2개 이상 브로커 정보를 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는데 이슈가 없도록 설정 가능하다.
- application.id 스트림즈 애플리케이션을 구분하기 위한 고유한 아이디를 설정한다. 다른 로직을 가진 스트림즈 어플리케이션은 서로다른 id를 가진다.
- 선택옵션
    - default.key.serde: 레코드의 메시지 키를 직렬화, 역질렬화하는 클래스를 지정한다. 기본 값은 바이트 직렬화
    - default.value.serde: 레코드의 메시지 값을 직렬화, 역질렬화하는 클래스를 지정한다. 기본 값은 바이트 직렬화
    - num.stream.threads: 스트림 프로세싱 실행 시 실행될 스레드 개수를 지정한다. 기본값 1.
    - state.dir: 상태기반 데이터 처리를 할 때 데이터를 저장할 때 디렉토리를 지정. 기본 값은 /tmp/kafka-streams.

### 실행
- bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic stream_log
- bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic stream_log_filter

```java
public class StreamsFilter {


    private static String APPLICATION_NAME = "streams-filter-application";
    private static String BOOTSTRAP_SERVER = "my-kafka:9092";
    private static String STREAM_LOG = "stream_log";
    private static String STREAM_LOG_FILTER = "stream_log_filter";

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, APPLICATION_NAME);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVER);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        StreamsBuilder builder = new StreamsBuilder();
        KStream<String, String> streamLog = builder.stream(STREAM_LOG);

        streamLog.filter((k, v) -> v.length() > 5).to(STREAM_LOG_FILTER);

        KafkaStreams streams;
        streams = new KafkaStreams(builder.build(), props);
        streams.start();
    }
}

```

## KTable, KStream을 join()
- KTable, KStream은 메시지 키를 기준으로 조인할 수 있다. 대부분의 데이터베이스는 정적으로 저장된 데이터를 조인하여 사용했지만, 카프카는 실시간으로 들어오는 데이터들을 조인할 수 있다.
- 사용자의 이벤트 데이터를 디비에 저장하지 않고 조인하여 스트리밍처리 할 수 있다.




## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D