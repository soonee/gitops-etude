# Commit Message 작성 기준

commit message는 짧지만 구체적이어야 한다.

좋은 commit message는 "무엇을 건드렸는가"보다 "어떤 의도의 변경인가"를 드러낸다.

## Summary

summary는 한 줄로 변경 의도를 요약한다.

좋은 예시는 아래와 같다.

```text
docs(worktree): explain why branch alone is not enough
fix(remote): prevent push to wrong default remote
refactor(history): split cleanup steps by risk
```

피해야 할 예시는 아래와 같다.

```text
fix: fix bug
docs: update docs
refactor: cleanup
```

무엇을 고쳤는지보다 어떤 의도로 바꿨는지가 보여야 한다.

## Scope

`scope`는 변경 범위를 좁혀 보여주는 선택 요소다.

```text
docs(worktree): add prune command explanation
docs(commit): split prefix and writing rules
fix(remote): set push default per worktree
```

scope는 모듈명, 문서명, 도메인명, 기능명을 쓸 수 있다. 애매하면 생략한다.

## Body

한 줄 summary만으로 충분하지 않다면 본문을 추가한다.

```text
fix(remote): prevent push to wrong default remote

Each worktree now sets remote.pushDefault explicitly.
This reduces accidental pushes when multiple remotes exist in one repository.
```

본문에는 "무엇을 했는가"보다 아래 내용을 적는 편이 좋다.

- 왜 했는가
- 어떤 위험을 줄였는가
- 운영상 무엇이 달라졌는가
- 리뷰어가 알아야 할 전제는 무엇인가

## Commit 단위

commit은 너무 작아도, 너무 커도 읽기 어렵다.

좋은 단위는 아래 기준으로 판단한다.

- 하나의 의도를 가진다
- 되돌렸을 때 영향 범위가 명확하다
- summary만 봐도 변경 이유가 대략 보인다
- 리뷰할 때 한 번에 이해 가능한 크기다

작업 중에는 commit을 자유롭게 만들 수 있다. main에 반영하기 전에는 `git rebase -i`로 noisy commit을 의미 있는 단위로 정리하는 것이 좋다.

## Rebase와 Commit Message

feature branch를 main에 병합하기 전에는 commit message도 함께 정리한다.

예를 들어 아래 commit들은 작업 중에는 자연스럽다.

```text
wip
fix typo
try another approach
cleanup
```

하지만 main에 들어가기 전에는 아래처럼 읽히는 단위로 바꾸는 편이 좋다.

```text
feat(auth): add login form
feat(auth): handle token refresh
docs(auth): document session expiry behavior
```

commit message 정리는 Git 히스토리를 운영 문서처럼 읽히게 만드는 과정이다.
