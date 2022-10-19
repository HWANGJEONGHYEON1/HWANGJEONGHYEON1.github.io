---
layout: post
title:  "Google Cloud Platform CI/CD 교육"
date:   2022-10-18 18:20:21 +0900
categories: GCP
---

> GCP 교육 (DevOps CI/CD)

# 10/18 GOOGLE 방문 교육

## End-to-end Platform for Container
1. Cloud Code => 오픈소스
2. 빌드
3. 컨테이너 페키지에 저장
4. 배포
5. Cloud Run (GKE (쿠버네티스))
6. Cloud Monitoring / Logging
- 프로세스
    1. 번째 SourceCode -> dockerFile -> K8s Mannifests -> GKE 배포
    2. 번째 방법 SourceCode -> dockerFile -> Cloud Run
    3. 번째 방법 SourceCode -> Cloud Functions

## Container
- 일관성있는 환경 (Consistence Environment)
- 인프라의 변경없음 (Immutable infrastructure)
- (Run anywhere)
- (Portability)
- (Ease of Sharing)
- (Reusability)

## 쿠버네티스란 ?
- 스케쥴링 
    - 현재 클러스터 리소스를 쿠버네티스가 알고 있음.
- 헬스체크
- 스케일링(컨테이너는 프로세스 기반으로 생성)
    - 빠른 속도로 스케일 아웃이 됨
- 로드 벨런싱

## GKE(Google Kubernates Engine)
- 보안
- 마이그레이션
- 커스텀 머신타입
- 선점형 VM
- UI < 제공


## Autopilot of Google Kubernetes Engine
- Autopilot: a hands-off Kubernetes experience
● Optimize for production like a K8s expert
● Strong security posture
● Google is your SRE (Reduce day 2 ops)
● Improve resources efficiency
● It’s still Kubernetes, still GKE
- standard vs autopilot

## Cloud Run
- serverless
    - auto-scaling
    - no Infra Management
    - spped to market
- split traffic
    - 카나리 배포같은 기능

## CI/CD
