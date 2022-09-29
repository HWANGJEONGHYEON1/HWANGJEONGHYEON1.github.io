---
layout: post
title:  "[study] kafka - 2"
date:   2022-09-028 16:20:21 +0900
categories: kafka
---

> 카프카 스터디 2

## 카프카 클러스터를 운영하는 방법
- IaaS 또는 온프레미스 환경에서 카프카 클러스터를 운영하는것이 가장 흔한 방법.
- 카프카는 전송된 데이터를 모두 파일 시스템에 저장하고 대규모 데이터 통신이 일어나기 때문에 고성능의 하드웨어를 사용해야한다.

| 항목 | 개발용 카프카 클러스터 | 상용환경 카프카 클러스터 |
| 브로커개수 | 5개 | 10개 |
| 메모리 | 16GB(heap memory 6GB) | 32 GB(heap memory 6GB) |
| CPU | 16core | 24 core |
| DISK | 달라짐 | 달라짐 |

## 카프카 커멘드 라인 툴
- 커맨드 라인 툴을 통해 토픽 관련 명령을 실핼할 때 `필수옵션`과 `선택 옵션`이 있다.
- 선택옵션 : 지정하지 않을 시 브로커에 설정된 기본 설정값 또는 커맨드라인 툴의 기본 값으로 대체되어 설정된다.
- 브로커 옵션이 어떻게 설정되어있는지 확인 후에 사용하면 커맨드라인 툴 사용 시 실수할 확률이 줄어든다.

## 카프카 바이너리
- bin
    - 실행할 shell script, 브로커 실행
- config
    - 설정에 필요한 여러 설정파일
- libs
    - 라이브러리

## kafka-topics.sh
- 데이터처리량을 더 늘리려면 간단하게 파티션을 늘리면 되지만, 줄일 수는 없다.
    - InvalidPationException
- 분산 시스템에서 이미 분산된 데이터를 줄이는 방법은 매우 복잡. 삭제 대상 파티션을 지정해야할 뿐만 아니라 기존에 저장되어있던 레코드를 분산하여 저장하는 로직이 필요
    - 카프카는 제공하지 않는다.
    - `새로 토픽을 만들자`

~~~
bin/kafka-topics.sh -- create \
 --bootstrap-server my-kafka:9092 \
 --topic hello.kafka

 bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --partitions 10 --replication-factor 1 --topic hi.kafka2 --config retention.ms=172800000
~~~

## kafka-configs.sh
- min.insync.replicas
~~~
 ✘ aa  ~/study/kafka/kafka_2.12-2.5.0  bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --describe
Topic: test	PartitionCount: 5	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
 we  ~/study/kafka/kafka_2.12-2.5.0  bin/kafka-configs.sh --bootstrap-server my-kafka:9092 --alter --add-config min.insync.repl
icas=2 --topic test
Completed updating config for topic test.
 aa  ~/study/kafka/kafka_2.12-2.5.0  bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --describe
Topic: test	PartitionCount: 5	ReplicationFactor: 1	Configs: min.insync.replicas=2,segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 4	Leader: 0	Replicas: 0	Isr: 0
~~~

## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D