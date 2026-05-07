# Branch Rebase

이 문서는 feature branch에서 rebase를 사용하는 이유와 rebase가 commit에 어떤 영향을 주는지 정리한다.

## Rebase는 무엇을 해결하는가

feature branch가 오래 열려 있으면 main은 계속 앞으로 간다.

```text
A---B---C main
     \
      D---E feature
```

이때 feature를 최신 main 위로 rebase하면 아래처럼 된다.

```text
A---B---C main
         \
          D'---E' feature
```

여기서 `D'`, `E'`는 기존 `D`, `E`와 내용은 비슷하지만 새로 만들어진 commit이다. parent가 바뀌었기 때문에 commit hash도 바뀐다.

## Feature Branch에서 Rebase가 좋은 이유

rebase가 유리한 이유는 아래와 같다.

- main 기준 최신 변경 위에서 충돌을 미리 해결할 수 있다
- main에 들어갈 때 히스토리가 직선에 가깝게 유지된다
- 중간의 실험 commit, 오타 수정 commit, 되돌림 commit을 정리할 수 있다
- PR이나 코드 리뷰에서 변경 의도가 더 잘 보인다

즉, feature branch에서 rebase를 쓰는 목적은 단순히 병합을 쉽게 하려는 것이 아니라, main에 들어갈 변경 이력을 읽을 수 있는 단위로 만드는 데 있다.

## 의미 있는 Commit 단위로 재구성하기

작업 중에는 commit을 자유롭게 해도 된다.

```text
feat: start login page
fix: typo
wip
fix again
refactor
```

하지만 main에 들어가기 전에는 이런 commit을 그대로 둘 필요가 없다. 이때 `git rebase -i`로 commit을 의미 있는 단위로 재구성한다.

```bash
git fetch origin
git rebase -i origin/main
```

여기서 하는 일은 "재커밋"이라기보다 "히스토리 재작성" 또는 "commit 재구성"이라고 부르는 편이 정확하다.

정리 후에는 아래처럼 남기는 것이 좋다.

```text
feat(auth): add login form
feat(auth): connect token refresh flow
docs(auth): document session expiry behavior
```

commit 하나하나가 main에서 읽혔을 때도 의미가 있어야 한다.

## Rebase와 메타정보

`rebase`는 기존 commit을 그대로 옮기는 것이 아니라, 새 parent 위에서 commit을 다시 만든다.

따라서 아래는 바뀐다.

- commit hash
- parent commit
- committer 정보
- committer date

기본적으로 아래는 유지된다.

- author name
- author email
- author date
- commit message

여기서 `author`는 한국어로 보통 `작성자` 또는 `저자`라고 부른다. 이 문서에서는 `작성자(author)`를 기본 표현으로 쓴다.

`committer`는 자연스러운 완전 번역어가 덜 굳어져 있어서 보통 `커미터(committer)`라고 부른다. 풀어 쓰면 `커밋 반영자` 또는 `커밋 수행자`에 가깝다.

정리하면 아래처럼 이해하면 된다.

- `author`
  원래 변경을 만든 사람
- `committer`
  그 변경을 실제 commit object로 기록한 사람

rebase 후에는 원래 변경의 작성자는 유지되지만, 새 commit object를 만든 사람은 rebase를 수행한 사람이 된다.

commit 메타정보 자체는 [../commit/metadata.md](../commit/metadata.md)에서 별도로 정리한다.
