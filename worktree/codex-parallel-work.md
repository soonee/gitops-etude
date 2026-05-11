# Codex Fork와 병렬 작업에서 Worktree 쓰기

`codex fork`로 같은 대화 세션에서 여러 작업을 동시에 시작하면 대화의 출발점은 같아진다. 하지만 파일시스템까지 자동으로 분리되는 것은 아니다.

따라서 같은 저장소 디렉터리에서 여러 Codex 세션을 동시에 돌리면 아래 문제가 생길 수 있다.

- 서로 같은 파일을 동시에 수정한다
- 한 세션이 만든 변경을 다른 세션이 예상하지 못한 상태로 읽는다
- 아직 commit하지 않은 변경이 섞인다
- `git status`, 테스트 결과, 빌드 산출물이 어떤 작업의 결과인지 모호해진다
- 마지막에 적용된 수정이 앞선 수정을 덮어쓸 수 있다

이때 `git worktree`를 쓰면 각 Codex 세션에 별도의 작업 디렉터리와 branch를 줄 수 있다.

## 역할 구분

`codex fork`, branch, worktree의 역할은 서로 다르다.

- `codex fork`는 대화 컨텍스트를 복제한다
- branch는 변경 이력을 나눈다
- `worktree`는 작업 디렉터리를 나눈다

즉, Codex 병렬 작업에서 안전한 기본 단위는 아래처럼 잡는 것이 좋다.

```text
작업 하나 = Codex 세션 하나 = worktree 하나 = branch 하나
```

이렇게 해야 에이전트들이 같은 출발점에서 시작하더라도 서로의 미완성 파일 상태를 직접 밟지 않는다.

## 권장 흐름

기본 저장소가 `/Volumes/workspace/soonee/gitops-etude`라고 가정한다.

먼저 기준 branch 상태를 깨끗하게 만든다.

```bash
cd /Volumes/workspace/soonee/gitops-etude
git status --short
git fetch --all --prune
```

그다음 작업별로 sibling 디렉터리에 worktree를 만든다.

```bash
git worktree add -b agents/worktree-doc ../gitops-etude-agent-worktree-doc main
git worktree add -b agents/commit-doc ../gitops-etude-agent-commit-doc main
git worktree add -b agents/roadmap-doc ../gitops-etude-agent-roadmap-doc main
```

worktree는 기준 저장소 내부가 아니라 형제 디렉터리로 만드는 편이 안전하다. 저장소 내부에 또 다른 worktree 디렉터리를 만들면 untracked 파일처럼 보이거나, 도구가 하위 디렉터리를 같은 프로젝트 일부로 오해할 수 있다.

각 worktree에서 별도 Codex 세션을 시작한다.

```bash
codex fork --last --cd ../gitops-etude-agent-worktree-doc "worktree 병렬 작업 문서를 보강해줘"
codex fork --last --cd ../gitops-etude-agent-commit-doc "commit 문서를 보강해줘"
codex fork --last --cd ../gitops-etude-agent-roadmap-doc "ROADMAP 문서를 정리해줘"
```

이 구조에서는 세 Codex 세션이 같은 대화 출발점을 공유하더라도 실제 파일 수정은 서로 다른 디렉터리와 branch에서 일어난다.

## 왜 같은 디렉터리에서 그냥 Fork하면 위험한가

`codex fork`는 Git branch를 자동으로 만들거나 작업 디렉터리를 복제하는 명령이 아니다. 같은 `--cd` 경로에서 fork한 세션을 여러 개 실행하면, 여러 에이전트가 같은 working tree를 공유한다.

이 상태에서는 사람이 동시에 같은 디렉터리에서 편집기를 여러 개 띄워 놓고 작업하는 것과 비슷하다.

- Git은 미완성 변경의 소유자를 구분하지 않는다
- Codex 세션끼리 서로의 계획을 자동으로 조율하지 않는다
- 같은 파일을 수정하면 마지막 수정이 앞선 수정을 덮을 수 있다
- 한 세션이 테스트를 돌리는 동안 다른 세션이 파일을 바꾸면 결과 해석이 어려워진다

따라서 Codex 세션을 병렬로 돌릴수록 작업공간 격리가 중요해진다.

## Worktree가 해결하는 것과 해결하지 못하는 것

`worktree`가 해결하는 것은 작업 디렉터리 격리다.

- 각 세션의 `git status`가 분리된다
- 미완성 변경이 다른 세션에 섞이지 않는다
- branch별 테스트, 빌드, 실행 상태를 따로 유지할 수 있다
- 작업을 버리거나 제거할 때도 해당 worktree만 정리하면 된다

하지만 `worktree`가 merge conflict까지 없애 주는 것은 아니다.

두 세션이 서로 다른 worktree에서 같은 파일을 수정했다면, 마지막에 branch를 합칠 때 conflict가 날 수 있다. 이 conflict는 나쁜 것이 아니라 Git이 같은 줄의 변경을 자동으로 판단할 수 없다는 신호다.

즉, `worktree`는 "동시에 안전하게 작업하게 해주는 도구"이지, "동시에 한 파일을 마음대로 수정해도 자동으로 합쳐 주는 도구"는 아니다.

## 병렬 작업 설계 기준

Codex 병렬 작업을 시작하기 전에는 작업 소유 범위를 먼저 나누는 편이 좋다.

좋은 분리 예시는 아래와 같다.

- 한 세션은 `worktree/` 문서만 수정한다
- 한 세션은 `commit/` 문서만 수정한다
- 한 세션은 테스트 실패 원인만 조사한다
- 한 세션은 코드 변경 없이 리뷰만 수행한다
- 한 세션만 `README.md`, `ROADMAP.md` 같은 공통 인덱스를 마지막에 정리한다

위험한 분리 예시는 아래와 같다.

- 여러 세션이 동시에 `README.md`를 수정한다
- 여러 세션이 같은 모듈의 같은 파일을 수정한다
- 한 세션이 대규모 리팩터링을 하고, 다른 세션이 같은 영역에 기능을 추가한다
- 공통 설정 파일을 여러 세션이 동시에 고친다

공통 파일 수정이 필요하다면 병렬 작업자가 직접 고치기보다 마지막 통합 세션이 한 번에 정리하는 편이 안전하다.

## 같은 파일을 수정해야 한다면

여러 세션이 같은 파일을 반드시 수정해야 한다면, 그것은 worktree로 완전히 해결할 수 있는 문제가 아니다. 이 경우에는 작업공간 격리보다 통합 순서 설계가 더 중요하다.

안전한 선택지는 아래 순서로 판단한다.

- 같은 파일 수정이 필요 없는 단위로 작업을 다시 나눈다
- 한 세션만 해당 파일의 소유자가 되고, 다른 세션은 그 파일을 건드리지 않는다
- 먼저 끝난 branch를 main에 반영하고, 다음 branch를 최신 main 위로 rebase한다
- 충돌이 예상되는 파일은 마지막 통합 세션에서 한 번에 정리한다
- 같은 파일의 같은 영역을 바꾸는 작업은 병렬화하지 않는다

즉, worktree는 병렬 작업의 안전장치지만 설계 없이 생긴 의미 충돌까지 없애지는 못한다.

## 통합 흐름

각 worktree 작업이 끝나면 해당 branch에서 commit을 만든다.

```bash
cd ../gitops-etude-agent-worktree-doc
git status --short
git add worktree/codex-parallel-work.md worktree/README.md ROADMAP.md
git commit -m "docs(worktree): explain codex parallel workflow"
```

기준 저장소로 돌아와 하나씩 통합한다.

```bash
cd /Volumes/workspace/soonee/gitops-etude
git switch main
git merge --ff-only agents/worktree-doc
```

다른 branch가 이미 main에 들어갔다면, 나머지 작업 branch는 최신 main 위로 다시 올린 뒤 통합한다.

```bash
cd ../gitops-etude-agent-commit-doc
git fetch --all --prune
git rebase main
```

충돌이 나면 그 시점에 해결한다. 충돌 해결 후에는 다시 테스트하고 commit을 정리한 뒤 main에 반영한다.

## Worktree 제거

작업이 끝난 worktree는 기준 저장소에서 제거한다.

```bash
cd /Volumes/workspace/soonee/gitops-etude
git worktree remove ../gitops-etude-agent-worktree-doc
git branch -d agents/worktree-doc
```

수동으로 디렉터리를 지웠다면 아래 명령으로 Git의 남은 연결 정보를 정리한다.

```bash
git worktree prune
```

## 결론

Codex 병렬 작업에서 `codex fork`만 쓰면 대화는 나뉘지만 작업공간은 나뉘지 않는다.

안전한 병렬 작업을 하려면 `codex fork`와 `git worktree`를 같이 써야 한다. `codex fork`는 같은 판단 맥락을 복제하고, `git worktree`는 각 작업자의 파일 상태와 branch를 격리한다.
