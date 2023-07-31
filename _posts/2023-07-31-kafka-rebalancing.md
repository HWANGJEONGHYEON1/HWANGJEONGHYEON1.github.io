---
layout: post
title:  "카프카 리벨런싱"
date:   2023-07-31 23:20:21 +0900
categories: kafka
---

> 카프카 리벨런싱 테스트

프로듀싱 된 데이터가 있을 때
컨슈머 구동 시

```
컨슈머 생성 시
[2023-07-31 10:28:30,615] INFO [GroupCoordinator 0]: Preparing to rebalance group group-01 in state PreparingRebalance with old generation 10 (__consumer_offsets-45) (reason: Adding new member consumer-group-01-1-9034960d-23e5-43e8-b3d4-93658400a22a with group instance id None) (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:28:33,213] INFO [GroupCoordinator 0]: Stabilized group group-01 generation 11 (__consumer_offsets-45) with 3 members (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:28:33,216] INFO [GroupCoordinator 0]: Assignment received from leader consumer-group-01-1-cfe2a223-cbaf-4748-b1cd-136595050b93 for group group-01 for generation 11. The group has 3 members, 0 of which are static. (kafka.coordinator.group.GroupCoordinator)

컨슈머 제거 시
[2023-07-31 10:31:49,693] INFO [GroupCoordinator 0]: Preparing to rebalance group group-01 in state PreparingRebalance with old generation 11 (__consumer_offsets-45) (reason: Removing member consumer-group-01-1-9034960d-23e5-43e8-b3d4-93658400a22a on LeaveGroup) (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:31:49,693] INFO [GroupCoordinator 0]: Member MemberMetadata(memberId=consumer-group-01-1-9034960d-23e5-43e8-b3d4-93658400a22a, groupInstanceId=None, clientId=consumer-group-01-1, clientHost=/127.0.0.1, sessionTimeoutMs=45000, rebalanceTimeoutMs=300000, supportedProtocols=List(range, cooperative-sticky)) has left group group-01 through explicit `LeaveGroup` request (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:31:51,322] INFO [GroupCoordinator 0]: Stabilized group group-01 generation 12 (__consumer_offsets-45) with 2 members (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:31:51,325] INFO [GroupCoordinator 0]: Assignment received from leader consumer-group-01-1-cfe2a223-cbaf-4748-b1cd-136595050b93 for group group-01 for generation 12. The group has 2 members, 0 of which are static. (kafka.coordinator.group.GroupCoordinator)

```

<b>왜 일어나는가?</b>

컨슈머 그룹내에 새로운 컨슈머가 추가 또는 제거되거나 또는 토픽에 새로운 파티션에 추가될 때 브로커의 Group coordinator는 컨슈머 그룹내의 컨슈머 들에게 파티션을 재할당하는 리벨런싱 수행하도록 지시

session.timeout.ms 이내에 하트비트 응답이없거나, max.poll.interval.ms 이내에 poll 메소드가 호출되지 않을 경우

Group coordinator : 컨슈머들의 조인 그룹 정보, 파티션 매핑정보 관리, 컨슈머들의 하트비트관리 

<i>과정</i>
1. 컨슈머 그룹네의 컨슈머가 브로케어 접속 요청 시 그룹코디네이터 생성
2. 동일 그룹ID 여러 개의 컨슈머로 브로커의 그룹 코디네이터로 접속
3. 가장 빨리 그룹에 요청한 컨슈머에게 리더할당
4. 리더가 지정된 컨슈머는 파티션 할당전력에 따라 컨슈머들에게 파티션할당
5. 리더 컨슈머는 최종할당된 파티션 정보를 그룹 코디네이터에게 전달
6. 컨슈머들은 파티션에서 메시지를 읽음


<hr>

### 컨슈머 스태틱 그룹 멤버쉽
많은 컨슈머들이 가지는 컨슈머 그룹에서 리벨린성 과정에서 모든 컨슈머들이 리벨런싱을 수행하므로 많은 시간이 소몯되고 대량 데이터 처리 시 렉이 길어질 수 있다.
유지보수 차원의 컨슈머 리스타트도 리벨런싱을 초래하므로 불필요한 리벨런싱을 방지

방법
- 컨슈머 그룹네의 컨슈머들에게 고정된 id를 부여
- 컨슈머 별로 컨슈머 그룹 최초 조인 시 할당된 파티션을 그대로 유지하고 컨슈머가 shutdown 되어도 session.timeout.ms 내에 재 기동될 시 리벨린성이 안되고 기존 파티션에 할당됨

session.timeout.ms = 45000
~~~
잘 기동된 후 컨슈머를 죽이고 재 기동해도 리벨런싱이 되지 않음 (45초 이전에 재기동)

[2023-07-31 10:47:48,438] INFO [GroupCoordinator 0]: Static member with groupInstanceId=3 and unknown member id joins group group-01-static in Stable state. Replacing previously mapped member 3-ef84a775-daba-4dfd-8208-f779d284595e with this groupInstanceId. (kafka.coordinator.group.GroupCoordinator)
[2023-07-31 10:47:48,444] INFO [GroupCoordinator 0]: Static member which joins during Stable stage and doesn't affect selectProtocol will not trigger rebalance. (kafka.coordinator.group.GroupCoordinator)

컨슈머가 죽은지 45초 이후 리벨런싱이 일어남 

이전
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                            HOST            CLIENT-ID
group-01-static test-topic     1          71              72              1               2-e850db2a-476d-4772-a033-a1b79bbe3ddd /127.0.0.1      consumer-group-01-static-2
group-01-static test-topic     0          66              70              4               1-c7df0578-3363-47d1-b169-05f977035f88 /127.0.0.1      consumer-group-01-static-1
group-01-static test-topic     2          42              47              5               3-ef84a775-daba-4dfd-8208-f779d284595e /127.0.0.1      consumer-group-01-static-3
 ~/study/kafka/confluent-7.1.3  bin/kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group group-01-static


이후 
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID                            HOST            CLIENT-ID
group-01-static test-topic     0          257             257             0               1-c7df0578-3363-47d1-b169-05f977035f88 /127.0.0.1      consumer-group-01-static-1
group-01-static test-topic     1          292             294             2               1-c7df0578-3363-47d1-b169-05f977035f88 /127.0.0.1      consumer-group-01-static-1
group-01-static test-topic     2          154             156             2               2-e850db2a-476d-4772-a033-a1b79bbe3ddd /127.0.0.1      consumer-group-01-static-2
~~~

### Heartbeat
heartbeat.interval.ms: 기본(3000ms) haertbeat 보내는 간격, session.timeout.ms의 1/3보다 낮게 권장 
session.timeout.ms: 기본(45000ms) 브로커가 consumer가 heartbeat을 기다리는 최대시간, 브로커는 이 시간동안 하트비트를 컨슈머로부터 받지 못하면 컨슈머를 그룹에섲 제외하도록 리밸런싱
max.poll.interval.ms: 기본(300000ms) 이전 poll 호출 후 다음 호출까지 브로커가 기다리는 시간, 해당 시간동안 poll 호출이 컨슈머로부터 이뤄지지 않으면 해당 컨슈머는 문제가 있는것으로 판단 후 브로커는 리벨린성 명령을 보냄 

