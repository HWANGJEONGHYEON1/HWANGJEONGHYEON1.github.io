---
layout: post
title:  "Effectivce Java - 3"
date:   2022-01-20 09:20:21 +0900
categories: java
---

> #2 기본에 충실하자 - 3

# 모든 객체의 공통 메서드

### item12) toString을 항상 재정의하라
- toString을 잘 구현한 클래스는 사용하기에 좋고, 디버깅이 쉽다.
- toString은 해당 객체에 관해 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야한다.

### item13) clone 재정의는 주의해서 진행하라
- 