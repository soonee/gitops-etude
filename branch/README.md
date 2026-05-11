# Git Branch

이 디렉터리는 branch를 단순히 `git branch` 명령이나 branch naming 규칙만의 문제가 아니라, 변경 흐름과 배포 흐름을 나누는 전략 단위로 다룬다.

branch를 이해할 때는 아래 질문을 함께 봐야 한다.

- feature branch는 main에 어떻게 반영하는가
- main과 배포 branch는 왜 더 보수적으로 다뤄야 하는가
- rebase, squash, merge commit은 언제 선택하는가
- branch 히스토리는 언제 rewrite해도 되는가

## 문서 목록

- [strategy.md](./strategy.md)
  feature branch, main, 배포 branch의 역할과 기본 운영 기준
- [rebase.md](./rebase.md)
  feature branch rebase, commit 재구성, rebase 시 메타정보 변화
- [rebase-boundary.md](./rebase-boundary.md)
  rebase가 replay할 commit 범위, `--onto`, stacked branch, 반복 rebase 전략
- [integration.md](./integration.md)
  merge, rebase 후 fast-forward, squash merge 선택 기준
- [release-branches.md](./release-branches.md)
  배포 branch를 보수적으로 운영해야 하는 이유
