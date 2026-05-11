# GitOps Foundations Roadmap

이 문서는 저장소 전체를 조망하기 위한 색인이다.

문서가 디렉터리 단위로 늘어나면 개별 문서는 읽기 좋아지지만, 전체 흐름이 흐려질 수 있다. 그래서 이 저장소는 각 디렉터리를 Git의 하위 명령 목록이 아니라 운영 질문 기준으로 배치한다.

## 전체 구조

```text
gitops-foundations
├── commit/
├── branch/
├── worktree/
├── history/
└── ROADMAP.md
```

## 개념 지도

### `commit`

commit은 Git 히스토리를 이루는 기본 기록 단위다.

여기서는 아래 질문을 다룬다.

- commit message를 어떻게 쓸 것인가
- commit 하나의 좋은 크기는 무엇인가
- author와 committer는 무엇이 다른가
- rebase, squash, cherry-pick은 commit을 어떻게 다시 만드는가

현재 문서:

- [commit/README.md](./commit/README.md)
- [commit/message-types.md](./commit/message-types.md)
- [commit/message-writing.md](./commit/message-writing.md)
- [commit/metadata.md](./commit/metadata.md)

### `branch`

branch는 변경 흐름을 나누는 전략 단위다.

여기서는 아래 질문을 다룬다.

- feature branch는 main에 어떻게 반영할 것인가
- rebase, squash, merge commit은 언제 선택할 것인가
- main과 배포 branch는 왜 rewrite하면 안 되는가
- GitOps 관점에서 배포 branch는 어떤 의미를 가지는가

현재 문서:

- [branch/README.md](./branch/README.md)
- [branch/strategy.md](./branch/strategy.md)
- [branch/rebase.md](./branch/rebase.md)
- [branch/integration.md](./branch/integration.md)
- [branch/release-branches.md](./branch/release-branches.md)

### `worktree`

worktree는 하나의 저장소를 여러 작업 공간으로 여는 운영 도구다.

여기서는 아래 질문을 다룬다.

- branch가 있는데 왜 worktree가 필요한가
- 여러 작업 컨텍스트를 동시에 유지하면 무엇이 달라지는가
- worktree별 Git identity와 push remote를 어떻게 분리할 것인가
- `codex fork` 같은 병렬 에이전트 작업에서 작업공간을 어떻게 격리할 것인가
- multi-remote 운영 사례와 어떻게 연결되는가

현재 문서:

- [worktree/README.md](./worktree/README.md)
- [worktree/concept.md](./worktree/concept.md)
- [worktree/lifecycle.md](./worktree/lifecycle.md)
- [worktree/config.md](./worktree/config.md)
- [worktree/multi-remote-case.md](./worktree/multi-remote-case.md)
- [worktree/codex-parallel-work.md](./worktree/codex-parallel-work.md)

### `history`

history는 공식 Git 객체 이름이 아니라, commit graph와 refs, reflog, GC까지 포함한 운영 분류다.

여기서는 아래 질문을 다룬다.

- `git rm --cached`는 왜 추적 중단일 뿐인가
- 과거 commit에서 파일을 제거하려면 무엇이 필요한가
- history rewrite 후에도 왜 서버나 clone에 흔적이 남을 수 있는가
- 민감정보가 commit됐다면 어떤 순서로 대응해야 하는가

현재 문서:

- [history/README.md](./history/README.md)
- [history/cleanup.md](./history/cleanup.md)

## 추천 읽는 순서

처음 읽는다면 아래 순서가 자연스럽다.

1. [commit/README.md](./commit/README.md)
2. [branch/README.md](./branch/README.md)
3. [worktree/README.md](./worktree/README.md)
4. [history/README.md](./history/README.md)

이 순서는 Git 내부 구조 순서가 아니라 운영 사고의 흐름이다. 먼저 commit 단위를 이해하고, branch로 변경 흐름을 나누고, worktree로 작업 공간을 나누고, 마지막으로 history rewrite와 cleanup의 위험을 다룬다.

## 앞으로 추가할 축

### `refs-and-index`

Git의 실제 움직임을 이해하려면 refs, HEAD, index, working tree를 따로 정리해야 한다.

후보 문서:

- `refs-and-index/refs.md`
- `refs-and-index/head.md`
- `refs-and-index/index.md`
- `refs-and-index/reset-restore-checkout.md`

### `remote`

GitOps로 가려면 remote와 source of truth를 분리해서 이해해야 한다.

후보 문서:

- `remote/README.md`
- `remote/upstream-and-push-default.md`
- `remote/multi-remote.md`
- `remote/refspec.md`

### `identity`

운영 환경에서는 누가 commit했고, 어떤 key로 push했는지가 중요하다.

후보 문서:

- `identity/git-config.md`
- `identity/include-if.md`
- `identity/ssh-key.md`
- `identity/signing.md`

### `gitops-bridge`

Git 자체의 이해를 GitOps 운영 모델로 연결한다.

후보 문서:

- `gitops-bridge/source-of-truth.md`
- `gitops-bridge/promotion.md`
- `gitops-bridge/drift.md`
- `gitops-bridge/repo-layout.md`

## 분류 기준

새 문서를 만들 때는 먼저 아래 질문으로 위치를 정한다.

- commit 하나의 의미와 기록 방식인가
- branch 흐름과 병합 전략인가
- 작업 공간을 나누는 문제인가
- 히스토리를 정리하거나 재작성하는 문제인가
- remote, refs, identity처럼 아직 별도 축이 필요한가

이 기준으로도 애매하면 단일 문서로 먼저 만들고, 내용이 커진 뒤 디렉터리로 승격한다.
