---
layout: post
title:  "Optimistic vs. Pessimistic Locking"
date:   2023-10-30 23:20:21 +0900
categories: spring, jpa, lock
---

> 블로그 글 읽고 요약 정리

### Pessimistic Lock
- 제로 데이터에 Lock 을 걸어서 정합성을 맞추는 방법입니다. exclusive lock 을 걸게되며 다른 트랜잭션에서는 lock 이 해제되기전에 데이터를 가져갈 수 없게됩니다. 데드락이 걸릴 수 있기때문에 주의하여 사용하여야 합니다.

### Optimistic lock
- 실제로 Lock 을 이용하지 않고 버전을 이용함으로써 정합성을 맞추는 방법입니다. 먼저 데이터를 읽은 후에 update 를 수행할 때 현재 내가 읽은 버전이 맞는지 확인하며 업데이트 합니다. 내가 읽은 버전에서 수정사항이 생겼을 경우에는 application에서 다시 읽은후에 작업을 수행해야 합니다

### 어느시점에 어느것을 사용?
- <b>Optimistic Locking</b>
  - 여러 트랜잭션 변경 내용을 커밋하려고 시도할 때까지 동시에 데이터에 대해 작업 가능
  - 비관적 잠금보다 잠금 오버헤드가 낮음
  - 잠금없이 동시에 읽기를 허용하므로 작업량이 많은 어플리케이션에 적합
- <b>Pessimistic lock</b>
  - 한 번에 하나의 트랜잭션만 데이터에 액세스할 수 있도록 보장하여 충돌을 방지
  - 데이터 일관성이 매우 중요할 때

### 예시
1. 이커머스 시스템
- Optimistic Locking
  - 주문을 확정하기전에 장바구니에 담은이후 상품의 재고가 변경되었는지 확인, 다른 유저가 동시에 구매하는 경우에 버전 불일치를 확인 후 주문 확인을 방지
- Pessimistic lock
  - 유저가 구매하기전에 재고를 lock을 걸고 `한명 만` 재고를 수정할 수 있도록 한다.

2. 좌석 예약 시스템
- Optimistic Locking
  - 두 명의 유저가 동시에 같은 자릴 요청 시, 예약 검증 스템에서 충돌을 감지, 모든 유저는 동시에 볼 수 있지만, 오버부킹되는 것을 방지 
- Pessimistic lock
  - 고객이 좌석을 예약할 수 있도록 허용하기 전에 시스템은 좌석을 독점적으로 잠그므로 한 번에 하나의 예약만 이루어질 수 있습니다. 이는 좌석이 이중 예약되지 않도록 보장

3. 뱅킹 시스템
- Optimistic Locking
  - 이체 중 시스템은 거래 전후에 계좌 잔액확인, 동시에 사용자가 이체를 하는경우, 한 사람만 성공할 수 있도록 허용, 버전 불일치를 감지하여 예외적으로 응답
- Pessimistic lock
  - 사용자가 이체할 때, 트랜잭션이 끝날 때까지 락을 잡고 있음.

4. 레스토랑 예약 시스템
- Optimistic Locking
  - 여러명의 웨이터가 오더를 테블릿으로 수정하려고 했을 때, 시스템은 충돌을 감지하고 수동해결
- Pessimistic lock
  - 웨이터가 오더를 시작하면 끝날 때까지 독점적으로 락을 잡고 있고, 다른 웨이터들과 동시에 수정하는것을 막음.


### 링크
- [참고 링크](https://dip-mazumder.medium.com/concurrency-control-in-java-persistence-api-jpa-with-hibernate-optimistic-vs-d74bb50fe4ec)