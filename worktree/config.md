# Worktree별 Git 설정

`worktree`마다 별도 설정을 사용하려면 먼저 아래 옵션을 켜야 한다.

```bash
git config extensions.worktreeConfig true
```

이후에는 각 디렉터리에서 `git config --worktree ...`로 개별 설정을 줄 수 있다.

여기서 중요한 점은 단순히 "`git config`를 바꿨다"가 아니라, 각 `worktree`의 `Git identity`와 기본 push 대상 remote를 분리한 것이라는 점이다.

`--worktree` 설정은 일반적인 repository config와 다르게 worktree별 설정 파일에 기록된다. 그래서 같은 저장소를 공유하더라도 worktree마다 `user.name`, `user.email`, `remote.pushDefault`를 다르게 유지할 수 있다.

## 용어 정리

이번 문서에서는 아래 표현을 기준으로 쓴다.

- `git config`는 설정을 조회하거나 변경하는 명령어 이름이다
- 실제로 바뀌는 대상은 `Git 설정`이다
- `user.name`, `user.email`은 `Git identity` 또는 작성자 설정이라고 부르는 편이 정확하다
- `remote.pushDefault`는 기본 push 대상 remote라고 표현하는 편이 자연스럽다

## 기본 Worktree

```bash
cd /path/to/repo-main

git config --worktree user.name "primary-author"
git config --worktree user.email "primary-author@example.invalid"
git config --worktree remote.pushDefault origin-main
```

## Profile A Worktree

```bash
cd /path/to/repo-profile-a

git config --worktree user.name "profile-a-author"
git config --worktree user.email "profile-a-author@example.invalid"
git config --worktree remote.pushDefault origin-profile-a
```

## Profile B Worktree

```bash
cd /path/to/repo-profile-b

git config --worktree user.name "profile-b-author"
git config --worktree user.email "profile-b-author@example.invalid"
git config --worktree remote.pushDefault origin-profile-b
```

이렇게 설정해 두면 각 디렉터리에서 단순히 `git push`만 실행해도 된다. 매번 `-C` 옵션으로 경로를 넘길 필요가 없다.
