# Branch 통합 방식

이 문서는 feature branch를 main에 반영할 때 `merge`, `rebase`, `squash`를 어떤 기준으로 선택할지 정리한다.

## Rebase 후 Fast-Forward

feature branch의 commit들이 각각 의미 있고, main 히스토리를 직선으로 유지하고 싶을 때 적합하다.

```bash
git checkout feature/auth
git rebase origin/main

git checkout main
git merge --ff-only feature/auth
```

이 방식은 깔끔하지만, feature branch의 개별 commit 품질이 좋아야 한다.

## Squash Merge

작업 중 commit이 많지만 main에는 하나의 기능 commit으로 남기고 싶을 때 적합하다.

```text
feat(auth): add login flow
```

작은 기능, 실험 commit이 많은 branch, 리뷰 중 수정 commit이 많은 branch에는 squash가 실용적이다.

단점은 feature branch 안의 세부 commit 이력이 main에서는 사라진다는 점이다.

## Merge Commit

feature branch가 큰 작업 단위이고, branch가 병합된 사실 자체를 히스토리에 남기고 싶을 때 적합하다.

```bash
git merge --no-ff feature/auth
```

다만 작은 feature가 많을 때 merge commit을 계속 남기면 main 히스토리가 빠르게 복잡해질 수 있다.

## 선택 기준

선택 기준은 아래처럼 잡으면 된다.

- commit 하나하나가 의미 있으면 rebase 후 fast-forward
- 작업 branch의 세부 이력이 중요하지 않으면 squash merge
- branch 단위의 병합 사실이 중요하면 merge commit
- 배포 branch나 장기 유지 branch에서는 추적 가능성을 우선

즉, 통합 방식은 취향이 아니라 히스토리를 어떻게 읽고 되돌릴 것인지에 대한 선택이다.
