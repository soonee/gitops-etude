# Git Worktree

이 디렉터리는 `git worktree`를 명령어 목록이 아니라 작업 컨텍스트 분리 전략으로 이해하기 위해 만든다.

`worktree`는 branch를 대체하는 기능이 아니다. 하나의 Git 저장소를 여러 작업 디렉터리로 동시에 열어 두게 해주는 기능이다.

## 문서 목록

- [concept.md](./concept.md)
  branch와 worktree의 차이, worktree가 필요한 상황
- [lifecycle.md](./lifecycle.md)
  worktree 생성, 제약, 목록 확인, 제거, 정리
- [config.md](./config.md)
  worktree별 Git 설정과 `Git identity`, 기본 push 대상 remote 분리
- [multi-remote-case.md](./multi-remote-case.md)
  여러 remote와 여러 identity를 worktree로 운영한 실제 사례
- [codex-parallel-work.md](./codex-parallel-work.md)
  `codex fork`로 같은 프로젝트를 병렬 작업할 때 worktree로 작업공간을 격리하는 방법

## 이 주제가 GitOps와 연결되는 이유

`worktree` 자체가 곧 GitOps는 아니다. 하지만 GitOps를 제대로 운영하려면 하나의 저장소를 어떤 단위로 나누고, 어떤 작업 컨텍스트를 동시에 유지하며, 어떤 변경을 어떤 경로로 전파할지에 대한 감각이 필요하다.

즉, 이 문서는 GitOps 도구를 다루기 전 단계에서 "Git을 운영 도구처럼 다루는 법"을 익히는 출발점이다.
