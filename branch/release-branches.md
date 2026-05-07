# Release Branch 운영 기준

배포 branch는 feature branch보다 훨씬 조심해야 한다.

배포 branch는 단순 개발 이력이 아니라 실제 운영 상태를 설명하는 기준이 될 수 있다.

## 배포 Branch가 가지는 의미

배포 branch는 아래 의미를 가질 수 있다.

- 특정 환경에 배포된 상태
- tag나 release note의 기준
- CI/CD 또는 GitOps 도구가 바라보는 source of truth
- 장애 대응과 rollback의 기준점

따라서 배포 branch에서는 히스토리 rewrite를 피하고, merge, revert, cherry-pick처럼 추적 가능한 방식이 더 적합한 경우가 많다.

## 왜 Rebase를 조심해야 하는가

GitOps 관점에서는 "무엇이 언제 어떤 환경에 반영됐는가"가 중요하다.

배포 branch를 rebase하면 commit hash와 parent가 바뀐다. 그러면 이미 배포된 상태를 가리키던 참조, release note, tag, 감사 기록, rollback 기준이 흔들릴 수 있다.

그래서 배포 branch는 히스토리를 예쁘게 만드는 것보다, 과거를 안정적으로 추적할 수 있는 것이 더 중요하다.

## 운영 기준

배포 branch에서는 아래 기준을 우선한다.

- force push를 피한다
- 이미 배포된 commit을 rewrite하지 않는다
- 문제가 있으면 revert commit으로 되돌린다
- 필요한 변경만 cherry-pick한다
- tag와 release note를 기준으로 상태를 추적한다

즉, feature branch는 정리하는 공간이고, 배포 branch는 기록을 보존하는 공간이다.
