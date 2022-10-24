---
layout: post
title:  "[study] kafka - 5"
date:   2022-10-24 18:20:21 +0900
categories: kafka
---

> 카프카 스터디 5

## 비교

### 카프카 스트림즈 vs 스파크 스트림즈
||kafka| spark|
|Development| Standalone java project| SparkExcutor|
|Streaming Source| kafka only| socket, file, kafka..|
|Exceution Model| Masterless|driver + excutor|
|Fault-Tolerance|StateStore, backed by changing| RDD Cache|
|Syntax|Low level Processor Api/ high Level DSL|Spark SQL|
|Semantics|Simple|Rich|

- 카프카 스트림즈: 카프카 토픽을 input 하는 경량 프로세싱 어플리케이션 개발
- 스파크 스트리밍: 카프카 토픽을 포함한 하둡 생태계 input인 복잡한 프로세싱 개발 


## 카프카 커넥트
- 데이터 파이프라인 생성 시 반복 잡업을 줄이고 효율적인 전송을 이루기 위한 어플리케이션이다.
- 커넥트는 특정한 작업 형태를 템플릿으로 만들어놓은 커넥터를 실행함으로써 효율적인 전송을 이루기 위한 어플리케이션이다.
- 내부구조
    - 사용자가 커넥트에 커넥터 생성명령을 내리면 커넥트는 내부에 커넥터와 테스크를 생성한다.
    - 커넥터는 태스크들을 관리. 테스크는 커텍터에 종속되는 개념으로 실질적인 데이터를 처리
    - 데이터 처리를 정상적으로 하는지 위해 테스크 상태를 확인해야한다.

### 소스 커넥터, 싱크 커넥터
- 프로듀서 역할 `소스 커넥터`
    - 파일의 데이터를 토픽으로 전송하는 역할
- 컨슈머 역할 `싱크 커넥터`
    - 토픽의 데이터를 파일로 저장하는 역할
- 커넥터 플러그인




## reference
- https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D