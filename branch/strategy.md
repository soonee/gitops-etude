# Branch 운영 전략

이 문서는 feature branch를 main 또는 배포 branch로 반영할 때의 기본 전략을 정리한다.

핵심은 "항상 rebase가 정답인가"가 아니라, 어떤 branch의 히스토리를 rewrite해도 되는지 구분하는 데 있다.

## 먼저 결론

feature branch는 main에 병합하기 전에 최신 main 위로 rebase하는 흐름이 좋은 기본값이다.

하지만 main이나 배포 branch 자체를 rebase하는 것은 보통 피한다. 이미 공유된 기준 branch의 히스토리를 rewrite하면 협업자, CI, 배포 추적, tag, GitOps 도구가 바라보는 기준이 흔들릴 수 있기 때문이다.

따라서 기본 전략은 아래처럼 잡는 것이 안전하다.

- feature branch에서는 rebase로 히스토리를 정리한다
- main 또는 배포 branch는 rewrite하지 않는다
- main에 들어가는 commit은 의미 있는 단위로 정리한다
- 이미 공유된 branch를 force push해야 한다면 `--force-with-lease`를 사용한다

## Feature Branch

feature branch는 보통 하나의 작업 주제를 담는다. 이 branch는 main에 들어가기 전에 정리해도 되는 임시 작업 공간에 가깝다.

feature branch에서는 아래 작업이 자연스럽다.

- 최신 main 위로 rebase
- `rebase -i`로 noisy commit 정리
- squash로 하나의 의미 있는 commit 만들기
- 의미 있는 commit 여러 개를 유지하기

즉, feature branch는 main에 들어갈 히스토리를 준비하는 공간이다.

## Main Branch

main은 여러 작업이 합쳐지는 기준 branch다.

main에서는 히스토리 안정성이 중요하다. 이미 공유된 main을 rebase하거나 force push하면 협업자의 로컬 히스토리와 원격 히스토리가 어긋난다.

따라서 main에서는 아래 기준을 우선한다.

- rewrite하지 않는다
- revert로 되돌릴 수 있게 한다
- CI와 release 기준을 안정적으로 유지한다
- 들어오는 commit의 품질을 feature branch에서 먼저 정리한다

## 배포 Branch

배포 branch는 main보다 더 보수적으로 다뤄야 한다.

배포 branch는 단순 개발 이력이 아니라 아래 의미를 가질 수 있다.

- 특정 환경에 배포된 상태
- tag나 release note의 기준
- CI/CD 또는 GitOps 도구가 바라보는 source of truth
- 장애 대응과 rollback의 기준점

따라서 배포 branch에서는 히스토리 rewrite보다 merge, revert, cherry-pick처럼 추적 가능한 방식이 더 적합한 경우가 많다.

## 운영 기준

이 저장소에서는 아래 기준을 기본으로 삼는다.

- feature branch는 main 병합 전 최신 main 위로 rebase한다
- main과 배포 branch는 rebase하지 않는다
- feature branch 안의 noisy commit은 `rebase -i`로 정리한다
- 의미 있는 commit 여러 개가 있다면 squash하지 않고 유지할 수 있다
- 의미 있는 단위가 하나라면 squash merge가 더 읽기 좋을 수 있다
- 배포 branch에서는 추적 가능성과 rollback 가능성을 우선한다

즉, feature branch에서는 rebase가 좋은 도구지만, main과 배포 branch에서는 히스토리 안정성이 더 중요하다.
