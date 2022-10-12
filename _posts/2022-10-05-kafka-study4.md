---
layout: post
title:  "[study] kafka - 3"
date:   2022-10-11 10:20:21 +0900
categories: kafka
---

> 카프카 스터디 4

## 실습 코드

### 커스텀 파티셔너를 가지는 프로듀서
- 프로듀서 환경에 따라 특정 데이터를 가지는 레코드를 특정 파티션으로 보내야할 때가 있다.
- 어떤 값은 반드시 특정 파티션에 들어가야한다고 했을 때 사용한다.
- Patitioner 인터페이스를 사용하여 할 수있다.

```java
public class CustomPartitioner implements Partitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {

        if (keyBytes == null) {
            throw new InvalidRecordException("Need message key");
        }

        if (key.equals("Pangyo")) {
            return 0;
        }

        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}


configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class);


```


### 레코드의 전송 결과를 확인하는 프로듀서
- KafkaProducer의 send() 메서드는 Future를 반환
- RecordMetadata의 비동기 결과를 표한하는 것으로 ProducerRecord가 카프카 브로커에 정상적으로 적재되었는지에 대한 데이터가 포함되어있다.
- get() 메서드를 사용하면 동기적으로 가져올 수 있다.

```java

		Properties configs = new Properties();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVER);
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
//        configs.put(ProducerConfig.ACKS_CONFIG, "0");

        KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
        int partitionNo = 0;
        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, partitionNo, "Pangyo", "판교");

        try {
            RecordMetadata metadata = producer.send(record).get();
            logger.info(metadata.toString());
        } catch (InterruptedException | ExecutionException e) {
            logger.error(e.getMessage(), e);
            throw new RuntimeException(e);
        } finally {
            producer.flush();
            producer.close();
        }

```

### 리벨런싱
- 컨슈머 그룹으로 이루어진 컨슈머 들 중 일부 컨슈머가 장애가 발생하면, 장애가 발생한 컨슈머에 할당된 파티션은 장애가 발생하지 않은 컨슈머에 소유권이 넘어간다. => `리벨런싱`
- 두 가지 상황
	- 컨슈머가 추가 되는 상황
	- 컨슈머가 제외 되는 상황
- 이슈가 발생한 컨슈머를 컨슈머 그룹에서 제외하여 모든 파티션이 지속적으로 데이터를 처리할 수 있도록 가용성을 높여준다.
- 컨슈머가 데이터를 처리하는 도중 언제든지 일어날 수 있으므로 리벨런싱 대응하는 코드가 있어야한다. 

### 커밋
- 컨슈머는 카프카 브로커로부터 데이터를 어디까지 가져갔는지 커밋을 통해 기록한다.
- 컨슈머 동작 이슈가 발생하여 _consumer_offsets 토픽에 어느레코드까지 읽어갔는지 오프셋 커밋이 못했다면 데이터 처리의 중복이 발생할 수 있다. 중복이 발생하지 않게 하기 위해 서는 에플리케이션 오프셋 커밋을 정상적으로 처리했는지 검증해야한다.

### assigner
- 컨슈머와 커밋의 할당 정책
- RangeAssigner : 각 파티션을 숫자로 정렬, 컨슈머를 사전 순서로 정렬하여 할당. (2.5.0 기본)
- RoundRobinAssigner: 모든 파티션을 컨슈머에서 번걸아가며 할당.
- StickyAssigner: 최대한 파티션을 균등하게 배분하면서 할당.

### 컨슈머 주요옵션
- bootstrap.servers: 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트이름:포트를 1개 이상 작성한다.
	- 2개 이상 브로커 정보를 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는데에 이슈가 없도록 설정 가능하다.
- key.deserializer: 레코드의 메시지 키를 역직렬화하는 클래스를 지정한다.
- value.deserializer: 레코드의 메시지 값를 역직렬화하는 클래스를 지정한다.

### 선택옵션
- groud.id : 컨슈머의 그룹 아이디를 지정한다. subscribe() 메서드로 토픽을 구독하여 사용할 때는 이 옵션을 필수로 넣어야한다. 기본 값은 Null
- auto.offset.reset: 컨슈머 그룹이 특정 파티션을 읽을 때 저장된 컨슈머 오프셋이 없는 경우 어느 오프셋부터 읽을지 선택하는 옵션이다. 기본값은 latest
- enable.auto.commit: 자동 커밋으로할지 수동 커밋으로할지 선택, 기본 값은 true
- auto.commit.interval.ms: 자동 커밋일 경우 오프셋 커밋 간격을 지정, 기본 값은 5초
- max.poll.records: poll() 메서드를 통해 반환되는 레코드 개수를 지정. 기본 값은 500
- session.timeout.ms: 컨슈머가 브로커와 연결이 끊기는 최대 시간이다. 기본 값은 10초
- heartbeat.interval.ms: 하트비트를 전송하는 시간  간격. 기본 값은 3초
- mx.poll.interval.ms: poll()메서드를 호출하는 간격의 최대시간. 기본 값은 5분
- isolation.level: 트랜잭션 프로듀서가 래코드를 트랜잭션 단위로 보낼 경우 사용한다.

### auto.offset.reset
- 컨슈머 그룹이 특정 파티션을 읽을 때 저장된 컨슈머 오프셋이 없는 경우 어느 오프셋부터 읽을지 선택하는 옵션. 이미 컨슈머 오프셋이 있다면 이 옵션값은 무시된다. 
- latest: 설정하면 가장 높은(최근) 오프셋부터 읽기 시작
- earliest: 설정하면 가장 작은 오프셋부터 읽기 시작
- none 설정하면 컨슈머 그룹이 커밋한 기록이 있는지 찾아본다. 만약 커밋기록이 없으면 오류를 반환하고, 커밋기록이 있다면 기존 커밋 기록 이후 오프셋부터 읽기 시작한다. 기본 값은 latest

## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D