---
layout: post
title:  "morningstudy-1"
date:   2022-01-13 08:20:21 +0900
categories: steady, blogging
---

> 매일 아침에 근무 전 1시간 반 공부 프로젝트(근무 시작 2시간 전 출근)

# 강의 정리(스프링)


### 로그 분석기 만들기
- 모든 public 메서드 호출과 응답 정보를 로그로 출력
- 어플리케이션의 흐름에 영향 x
- 메서드 호출 걸린 시간
- 정상흐름과 예외흐름 구분
- 메서드 호출의 깊이 표현
- http 요청구분
    - http 요청 단위로 특정 Id를 남겨서 어떤 http 요청에서 시작된 것인지 명확히 구분
    - http 시작부터 끝날 때까지 1개의 트랜잭션


```java

    private void complete(TraceStatus status, Exception e) { 
        Long stopTimeMs = System.currentTimeMillis();
        long resultTimeMs = stopTimeMs - status.getStartTimeMs(); TraceId traceId = status.getTraceId();
        if (e == null) {
            log.info("[{}] {}{} time={}ms", traceId.getId(),
                    addSpace(COMPLETE_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs);
        } else {
            log.info("[{}] {}{} time={}ms ex={}", traceId.getId(),
                    addSpace(EX_PREFIX, traceId.getLevel()), status.getMessage(), resultTimeMs, e.toString());
        } }

    private static String addSpace(String prefix, int level) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < level; i++) {
            sb.append( (i == level - 1) ? "|" + prefix : "| ");
        }
        return sb.toString();
    }

```