# Git Worktree로 작업 컨텍스트 분리하기

이 문서는 `gitops-foundations`의 첫 사례 문서다. `git worktree` 명령 전체를 설명하는 레퍼런스라기보다, 아래 질문에 답하기 위한 운영 문서에 가깝다.

- 브랜치가 있는데 왜 굳이 `worktree`가 필요한가
- 하나의 로컬 저장소를 여러 작업 컨텍스트로 나누면 무엇이 달라지는가
- `worktree`, 여러 `remote`, `Git identity` 분리를 어떻게 함께 운영할 수 있는가

즉, 이 문서의 관심사는 `git worktree` 자체보다 "하나의 저장소를 여러 관점으로 동시에 운영하는 패턴"에 있다.

## Worktree의 본질

`worktree`의 핵심은 브랜치를 새로 만드는 기능이 아니라, 하나의 Git 저장소를 여러 작업 디렉터리로 동시에 열어 둘 수 있게 해주는 기능이라는 점이다.

여기서 구분은 아래처럼 하는 것이 가장 명확하다.

- branch는 변경 이력을 나누는 논리적 단위다
- `worktree`는 작업 공간을 나누는 물리적 단위다

즉, `worktree`는 branch를 대체하지 않는다. 오히려 같은 저장소의 서로 다른 branch나 작업 컨텍스트를 동시에 유지할 수 있게 만들어 준다.

## 브랜치만으로 충분해 보이는데, 왜 Worktree가 필요한가

겉으로 보면 "하나의 로컬 저장소에서 브랜치를 만들고 `checkout`하면서 작업하면 되는 것 아닌가?"라는 질문이 자연스럽다. 이 질문은 맞다. 실제로 많은 경우에는 브랜치만으로도 충분하다.

핵심 차이는 branch 유무가 아니라 작업 디렉터리 개수에 있다.

- 브랜치만 쓰는 방식은 하나의 작업 디렉터리에서 branch를 순차적으로 바꿔 가며 작업한다
- `worktree`는 하나의 저장소에서 여러 작업 디렉터리를 동시에 유지한다

즉, 둘 다 branch는 쓴다. 차이는 "하나의 디렉터리에서 번갈아 바꿔 쓰느냐"와 "branch별 작업 공간을 동시에 열어 두느냐"에 있다.

### 브랜치만으로 충분한 경우

아래 상황이라면 굳이 `worktree`가 없어도 된다.

- 한 번에 하나의 branch에서만 작업한다
- 작업 트리가 자주 깨끗한 상태다
- 다른 branch를 잠깐 볼 때 `switch`, `checkout`, `stash`로 충분하다
- 브랜치 전환에 따른 재빌드, 재실행, 환경 전환 비용이 크지 않다

즉, 작업이 순차적이라면 일반 브랜치 전환만으로도 충분하다.

### Worktree가 유리한 경우

반대로 아래 상황에서는 `worktree`의 가치가 분명해진다.

- 아직 commit하지 않은 변경을 유지한 채 다른 branch의 긴급 수정을 해야 할 때
- 한쪽에서는 서버나 테스트를 계속 띄워 두고, 다른 쪽에서는 별도 작업을 해야 할 때
- 두 branch의 파일 상태를 나란히 비교해야 할 때
- 브랜치를 바꿀 때마다 `stash`, `checkout`, 재빌드, 재실행을 반복하는 비용이 클 때
- 같은 저장소를 서로 다른 목적의 작업 컨텍스트로 오래 유지하고 싶을 때

그래서 `worktree`는 branch를 더 쉽게 만드는 기능이 아니라, branch를 동시에 다룰 수 있게 해주는 기능이라고 이해하는 편이 정확하다.

## 이번 사례는 정확히 무엇을 한 것인가

이번 사례는 단순한 `worktree` 사용 예제라기보다 아래 세 가지가 결합된 운영 패턴이다.

- 하나의 로컬 저장소에 여러 `remote`를 등록했다
- 같은 로컬 저장소에서 여러 `worktree`를 만들었다
- 각 `worktree`마다 `Git identity`와 기본 push 대상을 분리했다

즉, 이번 작업을 한 문장으로 정리하면 아래 표현이 가장 적절하다.

> 하나의 로컬 저장소에 여러 remote를 등록하고, 여러 worktree를 만든 뒤, worktree별로 Git identity와 기본 push 대상 remote를 분리했다.

여기서 `worktree`가 중요한 이유는 단순히 branch를 더 만드는 데 있지 않다. 서로 다른 운영 습관을 물리적으로 분리해, 잘못된 author 정보나 잘못된 remote로 push하는 실수를 줄이는 데 있다.

## 이 사례에서 remote를 이해하는 최소 범위

이 저장소에서는 아직 `multi-remote`를 독립 주제로 분리하지 않았다. 현재 시점에서는 `multi-remote`가 `worktree` 사례를 구성하는 한 요소에 가깝기 때문이다.

이번 문서를 이해하는 데 필요한 remote 개념은 아래 정도면 충분하다.

- `remote`는 원격 저장소를 가리키는 이름과 URL 묶음이다
- 하나의 로컬 저장소는 여러 `remote`를 가질 수 있다
- `upstream`과 `remote.pushDefault`는 같은 의미가 아니다
- remote 이름이 다르다고 해서 역할이 자동으로 분리되지는 않는다

특히 헷갈리기 쉬운 두 개는 이렇게 구분하면 된다.

- `upstream`
  로컬 branch가 기본적으로 어떤 원격 추적 branch를 따라갈지와 관련된 개념이다
- `remote.pushDefault`
  옵션 없이 `git push` 했을 때 어디로 보낼지를 정하는 설정이다

즉, 이번 사례에서는 "여러 remote를 등록했다"보다 "worktree별로 기본 push 대상 remote를 분리했다"가 더 핵심이다.

## 용어 정리

이번 문서에서는 아래 표현을 기준으로 쓴다.

- `git config`는 설정을 조회하거나 변경하는 명령어 이름이다
- 실제로 바뀌는 대상은 `Git 설정`이다
- `user.name`, `user.email`은 `Git identity` 또는 작성자 설정이라고 부르는 편이 정확하다
- `remote.pushDefault`는 기본 push 대상 remote라고 표현하는 편이 자연스럽다

## 초기 생성 예시

```bash
cd /path/to/repo-main

git fetch --all --prune
git config extensions.worktreeConfig true

git worktree add -b publish/profile-a-master ../repo-profile-a master
git worktree add -b publish/profile-b-master ../repo-profile-b master
```

위 예시는 두 `worktree` 모두 현재 `master`가 가리키는 commit에서 시작한다.

- `git worktree add ... master`처럼 시작점을 명시하면 `master` 기준으로 생성된다
- 시작점을 생략하면 현재 `HEAD` 기준으로 생성된다

## Worktree별 설정

`worktree`마다 별도 설정을 사용하려면 먼저 아래 옵션을 켜야 한다.

```bash
git config extensions.worktreeConfig true
```

이후에는 각 디렉터리에서 `git config --worktree ...`로 개별 설정을 줄 수 있다.

여기서 중요한 점은 단순히 "`git config`를 바꿨다"가 아니라, 각 `worktree`의 `Git identity`와 기본 push 대상 remote를 분리한 것이라는 점이다.

### 기본 worktree

```bash
cd /path/to/repo-main

git config --worktree user.name "primary-author"
git config --worktree user.email "primary-author@example.invalid"
git config --worktree remote.pushDefault origin-main
```

### profile-a worktree

```bash
cd /path/to/repo-profile-a

git config --worktree user.name "profile-a-author"
git config --worktree user.email "profile-a-author@example.invalid"
git config --worktree remote.pushDefault origin-profile-a
```

### profile-b worktree

```bash
cd /path/to/repo-profile-b

git config --worktree user.name "profile-b-author"
git config --worktree user.email "profile-b-author@example.invalid"
git config --worktree remote.pushDefault origin-profile-b
```

이렇게 설정해 두면 각 디렉터리에서 단순히 `git push`만 실행해도 된다. 매번 `-C` 옵션으로 경로를 넘길 필요가 없다.

## 작업 흐름

원하는 흐름이 "루트에서 먼저 작업하고, 같은 변경을 다른 `worktree`에도 반영하는 방식"이라면 아래 순서가 기본 패턴이 된다.

### 1. 기본 worktree에서 작업 후 commit

```bash
cd /path/to/repo-main

git add .
git commit -m "feat: some change"
git push
```

### 2. profile-a worktree에 같은 변경 반영

```bash
cd /path/to/repo-profile-a

git cherry-pick -n <루트커밋SHA>
git commit -C <루트커밋SHA> --reset-author
git push
```

### 3. profile-b worktree에 같은 변경 반영

```bash
cd /path/to/repo-profile-b

git cherry-pick -n <루트커밋SHA>
git commit -C <루트커밋SHA> --reset-author
git push
```

## 명령 의미

- `git cherry-pick -n <SHA>`
  기존 commit의 변경 내용만 작업 트리에 적용하고, commit은 바로 만들지 않는다
- `git commit -C <SHA>`
  원래 commit 메시지를 재사용한다
- `git commit --reset-author`
  기존 author를 그대로 복사하지 않고, 현재 `worktree`의 `user.name`과 `user.email`을 author로 다시 기록한다

즉, author 정보는 `worktree`별 설정으로 덮어쓰는 방식으로 이해하면 된다. committer 역시 현재 작업 중인 `worktree` 기준으로 기록된다.

## Patch보다 Cherry-pick이 나은 이유

"루트에서 이미 commit을 만든 뒤, 다른 `worktree`에 같은 변경을 복제"하는 흐름이라면 `patch`보다 `cherry-pick -n`이 더 적합하다.

이유는 다음과 같다.

- commit 단위로 옮기므로 추적이 쉽다
- commit 메시지를 그대로 재사용하기 쉽다
- 임시 patch 파일을 만들고 관리할 필요가 없다

반대로 `patch`가 더 나은 경우는 아직 commit하지 않은 변경을 다른 작업 트리로 옮겨야 할 때 정도다.

## 이 주제가 GitOps와 연결되는 이유

`worktree` 자체가 곧 GitOps는 아니다. 하지만 GitOps를 제대로 운영하려면 하나의 저장소를 어떤 단위로 나누고, 어떤 작업 컨텍스트를 동시에 유지하며, 어떤 변경을 어떤 경로로 전파할지에 대한 감각이 필요하다.

즉, 이 문서는 GitOps 도구를 다루기 전 단계에서 "Git을 운영 도구처럼 다루는 법"을 익히는 기초 문서로 볼 수 있다.

정리하면, 이 문서는 `worktree`의 모든 기능을 다 설명하는 문서가 아니라 `gitops-foundations` 관점에서 `worktree`를 어떻게 이해하고 활용할지 보여주는 출발점이다.

## 다음에 이어질 주제

이 문서 다음에는 아래 주제가 자연스럽게 이어진다.

- `git-identity`
- `change-propagation`
- `branch-strategy`
- `gitops-bridge`
