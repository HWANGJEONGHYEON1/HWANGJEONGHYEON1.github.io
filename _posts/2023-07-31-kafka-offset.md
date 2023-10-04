---
layout: post
title:  "카프카 오프셋"
date:   2023-07-31 23:20:21 +0900
categories: kafka
---

> 카프카 오프셋 메카니즘

auto.enable.commit=true
 읽어온 메시지를 브로커에 바로 commit하지 않고, auto.commit.interval.ms에 정해준 주기마다 컨슈머가 자동으로 커밋을 수행
 컨슈머가 읽어온 메시지보다 브로커의 커밋이 오래되었으므로 컨슈머의 장애/재기동 및 리벨런싱 후 브로커에서 이미 읽어온 메시지를 다시 읽어와 중복처리 될 수도 있다.

컨슈머 그룹이 특정 토픽의 파티션별로 읽기 커밋한 오프셋 정보를 가짐, 특정 파티션을 어느 `컨슈머가 커밋했는지 정보를 가지지 않음`

중복된 오프셋 ?
상황 : 1번 커슈머가 데이터를 4~8번까지 읽고 디비에 저장, 그런데 커밋을 하지 못하고 컨슈머 1 죽음, 리벨런싱이 일어나 컨슈머2가 다시 4~8번까지 읽음 
