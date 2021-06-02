---
layout: post
title:  "git confilct 해결하기"
date:   2021-06-01 20:22
categories: git
tags: [git, branch]
---

## git confilct 


### 상황 
`feature/security branch`와 `feature/shop branch`가 있는대, 각각 Pull Request에 대해 feature/shop 먼저 머지를 하고 난 뒤 feature/security 머지 시 `충돌`이 발생했다.

### 해결
- `master` 브랜치로 이동
- `feature/shop` 건에 대해 pull
- 다시 `feature/security` 이동
- 머지 시도
- __충돌__발생
- Intellij에서 확인 후 충돌 난 부분 수정(자기가 사용하는 editor 사용)
- 충돌 해결 후 다시 PR 요청

 ### Ref.
* <https://velog.io/@devmin/git-conflict-solution-basic>