# Git Commit

이 디렉터리는 commit을 단순히 `git commit` 명령이나 commit message만의 문제가 아니라, Git 히스토리를 구성하는 기본 단위로 다루기 위해 만든다.

commit을 이해할 때는 최소한 아래 주제를 함께 봐야 한다.

- commit message는 변경 의도를 어떻게 표현하는가
- commit은 어떤 메타정보를 가지는가
- author와 committer는 무엇이 다른가
- rebase, squash, cherry-pick은 commit을 어떻게 다시 만드는가

## 문서 목록

- [message-types.md](./message-types.md)
  `docs`, `feat`, `fix`, `refactor`, `release` 같은 prefix 기준
- [message-writing.md](./message-writing.md)
  summary, scope, body 작성 기준과 예시
- [metadata.md](./metadata.md)
  commit의 작성자, 커미터, 날짜, hash가 어떤 의미를 가지는지 정리

## 기본 메시지 형식

이 저장소의 기본 commit message 형식은 아래처럼 잡는다.

```text
<type>(<scope>): <summary>
```

예시는 아래와 같다.

```text
docs(worktree): explain branch and worktree difference
feat(auth): add token refresh flow
fix(api): handle empty response body
refactor(repo): split remote setup helper
release: prepare 1.2.0
```

`scope`는 선택이다. 변경 범위가 명확하면 쓰고, 애매하면 생략한다.

## 초기 기준

이 저장소에서는 우선 아래 prefix를 기본으로 사용한다.

```text
docs
feat
fix
refactor
release
```

필요해지면 아래를 추가한다.

```text
chore
test
ci
build
ops
```

초기에는 prefix를 많이 늘리는 것보다, 적은 prefix를 일관되게 쓰는 것이 더 중요하다.
