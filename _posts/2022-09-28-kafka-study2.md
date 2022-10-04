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

## kafka-console-producer.sh
- bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hi.kafka
	- key 값이 null 로 들어가며, 각 파티션에 라운드로빈으로 등록됨
- bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hi.kafka --property "parse.key=true" --property "key.separator=:"
	- key:value로 값이 저장되며, 각 파티션에 있지만 key 값이 존재하고, 중복 키로 들어가는 경우 같은 파티션에 들어간다.

## kafka-console.consumer.sh
- bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hi.kafka
	-  --from-beginning
		- 가장 오래된 데이터부터 모든 데이터를 보여준다
	- print.key=true key.separator="-" --from-beginning
		- key, value 형태로 값을 보이게한다.
	-  --from-beginning --max-messages 2
		- 2개까지 보인다.
	- --partition 0
		- 특정 파티션에 있는 데이터를 보ㅣ게한다. 

## 커밋데이터
- bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
~~~
__consumer_offsets
hi.kafka
hi.kafka2
test
~~~

## kafka-consumer-group.sh
- 컨슈머 그룹은 따로 생성하는 명령을 날리지 않고 컨슈머를 동작할 때 컨슈머 그룹이름을 지정하면 새로 생성된다.
- describe
	- 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지
	- 상태
	- 파티션 번호, 현재까지 가져간 레코드의 오프셋, 파티션 마지막 레코드의 오프셋, 컨슈머 렉, 컨슈머 ID, 호스트
- reset
	- --to-earliest : 가장 처음 오프셋(작은번호)으로 리셋
	- --to-latest : 가장 마지막
	- --to-current : 현시점 기준 오프셋
 	- --to-datetime : 특정 일시로 오프셋 리셋
	- --to-offset {long} : 특정 오프셋으로 리셋
	- --shift-by {+/- long} : 현재 컨슈머 오프셋에서 앞 뒤로 옮겨 리셋

~~~

kafka:9092 --group hello-group --describe

Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hi.kafka        0          15              15              0               -               -               -
 we  ~/study/kafka/kafka_2.12-2.5.0  bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --group hello-group --describe

Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hi.kafka        0          15              22              7               -               -               -
~~~

- consumer로 데이터를 봤을때 LAG은 0 이었지만, 밑에서 재조회했을 때는 producer로 데이터를 7개 넣은 상태이다. 그래서 7이 증가하고 현재 데이터 처리량이 7개가 지연되고 있다라고 이해하면 된다.
- 데이터를 처음부터 다시 읽어야하는 상황이 필요하다면? reset


## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D