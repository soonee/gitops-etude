# Worktree와 Multi-Remote 운영 사례

이 문서는 `worktree`, 여러 `remote`, `Git identity` 분리를 함께 사용한 사례를 정리한다.

## 이번 사례는 정확히 무엇을 한 것인가

이번 사례는 단순한 `worktree` 사용 예제라기보다 아래 세 가지가 결합된 운영 패턴이다.

- 하나의 로컬 저장소에 여러 `remote`를 등록했다
- 같은 로컬 저장소에서 여러 `worktree`를 만들었다
- 각 `worktree`마다 `Git identity`와 기본 push 대상을 분리했다

즉, 이번 작업을 한 문장으로 정리하면 아래 표현이 가장 적절하다.

> 하나의 로컬 저장소에 여러 remote를 등록하고, 여러 worktree를 만든 뒤, worktree별로 Git identity와 기본 push 대상 remote를 분리했다.

여기서 `worktree`가 중요한 이유는 단순히 branch를 더 만드는 데 있지 않다. 서로 다른 운영 습관을 물리적으로 분리해, 잘못된 author 정보나 잘못된 remote로 push하는 실수를 줄이는 데 있다.

## 이 사례에서 remote를 이해하는 최소 범위

현재 시점에서는 `multi-remote`가 `worktree` 사례를 구성하는 한 요소에 가깝다.

이번 사례를 이해하는 데 필요한 remote 개념은 아래 정도면 충분하다.

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

## 작업 흐름

원하는 흐름이 "기본 worktree에서 먼저 작업하고, 같은 변경을 다른 `worktree`에도 반영하는 방식"이라면 아래 순서가 기본 패턴이 된다.

### 1. 기본 Worktree에서 작업 후 Commit

```bash
cd /path/to/repo-main

git add .
git commit -m "feat: some change"
git push
```

### 2. Profile A Worktree에 같은 변경 반영

```bash
cd /path/to/repo-profile-a

git cherry-pick -n <루트커밋SHA>
git commit -C <루트커밋SHA> --reset-author
git push
```

### 3. Profile B Worktree에 같은 변경 반영

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

"기본 worktree에서 이미 commit을 만든 뒤, 다른 `worktree`에 같은 변경을 복제"하는 흐름이라면 `patch`보다 `cherry-pick -n`이 더 적합하다.

이유는 다음과 같다.

- commit 단위로 옮기므로 추적이 쉽다
- commit 메시지를 그대로 재사용하기 쉽다
- 임시 patch 파일을 만들고 관리할 필요가 없다

반대로 `patch`가 더 나은 경우는 아직 commit하지 않은 변경을 다른 작업 트리로 옮겨야 할 때 정도다.
