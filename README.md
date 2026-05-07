# gitops-foundations

`gitops-foundations`는 GitOps 구현 저장소가 아니라, GitOps를 제대로 이해하고 운영하기 위해 먼저 필요한 Git의 심화 개념과 작업 패턴을 정리하는 저장소다.

즉, 이 저장소는 "Git 자체의 깊이"에서 출발해 "GitOps로 이어지는 기반"을 쌓는 흐름을 목표로 한다.

## 왜 이 이름인가

- `GitOps`는 Git을 단순한 형상관리 도구가 아니라 운영의 기준점으로 다루는 방식이다.
- 그러려면 branch, remote, refs, history 조작, identity, workspace 관리에 대한 이해가 먼저 필요하다.
- 그래서 이 저장소는 GitOps 자체보다 한 단계 아래의 기반 레이어를 다룬다.

## 문서 정리 원칙

- 문서는 "명령어 목록"보다 "운영 질문"을 기준으로 나눈다.
- 서로 강하게 겹치는 주제는 별도 문서로 쪼개지 않고 하나의 사례 문서로 합친다.
- 현재는 문서를 잘게 분해하기보다, 실제 운영 맥락이 보이도록 사례 중심으로 쓴다.
- 디렉터리로 문서를 나눌 때는 [ROADMAP.md](./ROADMAP.md)에서 전체 위치를 먼저 잡는다.

## 현재 문서

- [ROADMAP.md](./ROADMAP.md)
  전체 문서 구조, 읽는 순서, 앞으로 확장할 주제
- [worktree](./worktree/README.md)
  `git worktree`를 활용한 작업 컨텍스트 분리, 생명주기, worktree별 설정, multi-remote 운영 사례
- [branch](./branch/README.md)
  feature branch, rebase, squash, merge commit, 배포 branch 운영 기준
- [history](./history/README.md)
  `git rm --cached`, history rewrite, 민감정보 제거, reflog와 GC로 이어지는 히스토리 운영 주제
- [commit](./commit/README.md)
  commit message, commit metadata, author와 committer, commit 단위 정리

## 다루고 싶은 흐름

1. Git 자체의 구조와 동작 방식 이해
2. 실제 운영에서 쓰는 Git 작업 패턴 정리
3. 그 패턴이 GitOps와 어떻게 연결되는지 설명
4. 마지막에 GitOps 도구와 운영 모델로 확장

## 다음에 추가할 주제

- `refs-and-index.md`
- `git-identity.md`
- `change-propagation.md`
- `automation.md`
- `gitops-bridge.md`
- `case-study/`
