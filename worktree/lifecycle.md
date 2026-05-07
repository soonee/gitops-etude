# Worktree 생명주기

이 문서는 `worktree`를 만들고, 확인하고, 제거하고, 정리하는 기본 흐름을 다룬다.

## 초기 생성 예시

```bash
cd /path/to/repo-main

git fetch --all --prune
git worktree add -b publish/profile-a-master ../repo-profile-a master
git worktree add -b publish/profile-b-master ../repo-profile-b master
```

위 예시는 두 `worktree` 모두 현재 `master`가 가리키는 commit에서 시작한다.

- `git worktree add ... master`처럼 시작점을 명시하면 `master` 기준으로 생성된다
- 시작점을 생략하면 현재 `HEAD` 기준으로 생성된다

## 같은 Branch를 여러 Worktree에서 동시에 Checkout할 수 없다

`worktree`에서 가장 중요한 제약은 같은 branch를 여러 `worktree`에서 동시에 checkout할 수 없다는 점이다.

예를 들어 기본 worktree에서 이미 `master`를 checkout하고 있다면, 다른 worktree에서 다시 `master`를 checkout할 수 없다. Git은 같은 branch가 두 작업 디렉터리에서 동시에 수정되는 상황을 막기 위해 이 제약을 둔다.

그래서 새 `worktree`를 만들 때는 보통 아래처럼 별도 branch를 같이 만든다.

```bash
git worktree add -b publish/profile-a-master ../repo-profile-a master
```

이 명령은 `master`가 가리키는 commit에서 `publish/profile-a-master` branch를 만들고, 그 branch를 새 worktree에 checkout한다.

## 목록 확인

```bash
git worktree list
```

현재 저장소에 연결된 worktree 목록과 각 worktree가 checkout한 branch, commit을 확인한다.

## 제거

```bash
git worktree remove ../repo-profile-a
```

작업 디렉터리를 제거하면서 Git의 worktree 연결 정보도 함께 정리한다.

## 오래된 연결 정리

```bash
git worktree prune
```

수동으로 디렉터리를 지웠거나, 파일시스템 상태와 Git의 worktree 메타데이터가 어긋났을 때 남은 연결 정보를 정리한다.

즉, 가능하면 `rm -rf`로 직접 지우기보다 `git worktree remove`를 먼저 쓰는 편이 낫다. 이미 직접 지웠다면 `git worktree prune`으로 정리한다.
