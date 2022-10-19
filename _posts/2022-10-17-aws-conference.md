---
layout: post
title:  "AWS CI/CD 교육"
date:   2022-10-17 18:20:21 +0900
categories: kafka
---

> AWS 교육 (DevOps CI/CD)

# 10/17 AWS 본사 교육

## Agenda
- intro
- release process
- Amazone CI/CD
- Infrastructure as Codd
- Q&A

## DevOps란 ?
- 원칙
    - 지속적인 통합/지속적인 배포
    - 인프라 관리의 코드화/버전 관리 통합
    - 마이크로 서비스
    - 모니터링과 로깅
    - 의사소통/협업
- Release Process
    - Source => Build => Test => Production
    - Source
        - 개발자가 짠 코드를 동료에게 피드백을 받는다
    - Build
        - 코드 컴파일, 단위 테스트
        - 스타일 체크
    - Test
        - 통합 테스트
        - UI
        - Security
    - Production
        - 빠른 에러 감지

## 지속적 통합
- 개발자가 중앙저장소로 코드 변경 사항을 병합하는 소프트웨어 개발방식
- 버그를 더 빨리 찾고, 코드의 품질을 올린다.
- 빌드는 완전히 자동화
- 체크인 된 모든 코드를 테스트
- 개발자를 위한 즉각적인 피드백
    - 빌드가 빨라야한다.
- AWS CI/CD
    - Code Commit
        - flow
            - 개발자가 기능 브랜치 생성
            - PR 생성
            - PR 검토
            - PR 올린 브랜치가 병합이 됨
            - 기존 브랜치 삭제(옵션)
    - CodeGuru
        - Automated Review
            - java
            - python
    - Code Build
        - 완전 관리형 빌드 서비스
        - 빌드 및 테스트
        - 확장가능,보안-빌드
        - 도커 기반
    - Code Deploy
        - 완전 관리형 배포 서비스
        - 자동 배포
            - one at a time(차례차례)
            - half at a time (배포할 때 반만)
            - all at once
        - EC2, 온프레미스, 람다지원
        - 버전관리
        - 자동 확장과 통합
    - CodePipeline
        - 완전관리형 지속적 전달 서비스
        - 릴리스 프로세스 정의, 모델링, 자동화
        - 릴리스 품질 향상
        - 구성 가능한 워크플로우

## IaC (Infrastructure as Code)
- 인프라 구성요소를 생성, 구성 및 배포하기 위한 코드 작성
- 네트워킹, 컴퓨팅, 디비, 보안등이 포함
- 왜 필요한가
    - 인프라 변경 반복 및 예측가능
    - 인프라 문서화
    - 프로비저닝 프로세스 자동화
    - 자동화를 통해 구성 드리프트 제거

## 실습
- Cloud9 
- Code Commit
- Code Gruru
- CI/CD
    - blue/green
        - 새로운걸 만들어서 새로 옮긴다.

        


    