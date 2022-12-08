---
layout: post
title:  "스프링 JPA TEST 시 @Transactional"
date:   2022-12-07 18:20:21 +0900
categories: spring, jpa
---

> Spring JPA 서비스레이어 테스트 중 Update 쿼리가 발생하지 않는 문제 => `더블 트랜잭셔널`

## 스프링 @Transactional
- 이 어노테이션을 사용한이유는 디비에 데이터를 커밋, 롤백해주는 이유로 사용.
- 첫 트랜잭션이 시작되면 다음 트랜잭션을 만낫을 때 부모 트랜잭션을 이어서 받는것이 스프링의 기본 설정 값
- 트랜잭션이 커밋이 되는 순간 flush가 되어 쿼리가 나감. (<- 이걸 생각못함)

## 문제
- test 코드에 @Transactional을 걸어놓았는대, 
- 테스트 메소드의 트랜잭션을 걸어, update 트랜잭션이 종속되어 아직 커밋이 안되어 업데이트 쿼리가 나가지 않음.
- 히스토리에서 조회를 시도하지만, 아직 조회가 되지 않아 에러발생.

```java


    @Transactional
    public Long update(Long domainSeq, DomainRequestDto requestDto) {
        Domain domain = findByDomainSeq(domainSeq);

        checkValidRequest(requestDto);

        if (UseYn.N == domain.getUseYn()) {
            throw new BusinessException(ErrorCode.REQUEST_INVALID_R007);
        }

        domain.update(
                requestDto.getRouteId(),
                requestDto.getRouteUri(),
                requestDto.getFilters(),
                requestDto.getPredicates(),
                requestDto.getReason(),
                requestDto.getChgId()
        );

        return domainRepository.save(domain)
                .getId();
    }

    @Test
    @DisplayName("[수정 테스트] 도메인 수정")
    @Transactional
    @Tag(TagConstants.UPDATE)
    void update_domain() {
        // 등록
        Long domainSeq = domainService.create(DomainRequestHelper.createDto(testDomain, testUri));
        DomainRequestDto requestDto = DomainRequestDto.builder()
                .routeId("test999")
                .routeUri("https://test999.com")
                .predicates(testPredicates)
                .filters(testFilters)
                .reason("test999")
                .chgId("2021080000")
                .build();

        // 수정
        Long updateDomainSeq = domainService.update(domainSeq, requestDto);

        Domain updateDomain = domainRepository.findById(updateDomainSeq).get();
        Assertions.assertAll(
                () -> assertThat(updateDomain.getId()).isEqualTo(domainSeq),
                () -> assertThat(updateDomain.getRouteId()).isEqualTo("test999"),
                () -> assertThat(updateDomain.getRouteUri()).isEqualTo("https://test999.com"),
                () -> assertThat(updateDomain.getPredicates()).isEqualTo(testPredicates),
                () -> assertThat(updateDomain.getFilters()).isEqualTo(testFilters),
                () -> assertThat(updateDomain.getReason()).isEqualTo("test999"),
                () -> assertThat(updateDomain.getChgId()).isEqualTo("2021080000")
        );

        List<DomainHistory> updateDomainHistories = domainHistoryRepository.findByDomainSeq(updateDomainSeq, Sort.by(Sort.Direction.DESC, "id"));
        assertThat(updateDomainHistories.get(0).getHistoryType()).isEqualTo(HistoryType.UPDATE);
        assertThat(updateDomainHistories.get(1).getHistoryType()).isEqualTo(HistoryType.CREATE);
    }
```

## 해결
- 테스트의 @Transactional 어노테이션 제거
- @AfterEach 어노테이션을 통해 해당 데이터 제거 
  - 테스트 컨테이너를 사용해 실제 디비의 데이터와는 무관하다

```java
    @AfterEach
    void tearsDown() {
        domainHistoryRepository.deleteAll();
        domainRepository.deleteAll();
    }
```