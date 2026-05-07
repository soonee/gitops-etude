# Commit Metadata

commit은 파일 변경 내용만 저장하지 않는다. 누가 변경을 만들었는지, 누가 commit object로 기록했는지, 언제 만들어졌는지 같은 메타정보도 함께 가진다.

## Author와 Committer

Git commit에는 `author`와 `committer`가 따로 있다.

- `author`
  원래 변경을 만든 사람이다. 한국어로는 `작성자(author)`라고 부르는 편이 자연스럽다.
- `committer`
  그 변경을 실제 commit object로 기록한 사람이다. 한국어로는 보통 `커미터(committer)`라고 쓰고, 풀어 쓰면 `커밋 반영자` 또는 `커밋 수행자`에 가깝다.

일반적인 개인 작업에서는 작성자와 커미터가 같다. 하지만 rebase, cherry-pick, patch 적용, 협업자 commit 반영 같은 상황에서는 달라질 수 있다.

## 날짜 정보

commit에는 작성자 날짜와 커미터 날짜가 따로 있다.

- author date
  원래 변경이 작성된 시점
- committer date
  commit object가 현재 히스토리에 기록된 시점

rebase나 cherry-pick을 하면 기존 변경을 새 commit object로 다시 만들기 때문에 커미터 날짜가 새로 기록된다.

## Hash가 바뀌는 이유

commit hash는 변경 내용만 보고 정해지지 않는다. parent, author, committer, 날짜, message 같은 정보도 commit object의 일부다.

그래서 아래 작업은 commit hash를 바꿀 수 있다.

- rebase
- commit message 수정
- author 변경
- amend
- cherry-pick
- squash

즉, 내용이 같아 보여도 commit object가 다시 만들어지면 hash는 바뀐다.

## Rebase 때 유지되는 것과 바뀌는 것

`rebase`는 기존 commit을 그대로 옮기는 것이 아니라, 새 parent 위에서 commit을 다시 만든다.

기본적으로 유지되는 정보는 아래와 같다.

- author name
- author email
- author date
- commit message

바뀌는 정보는 아래와 같다.

- commit hash
- parent commit
- committer
- committer date

따라서 rebase 후에는 원래 변경의 작성자는 유지되지만, 새 commit object를 만든 사람은 rebase를 수행한 사람이 된다.

## 운영 관점

commit metadata는 감사, 배포 추적, 장애 대응, GitOps 흐름에서 중요해질 수 있다.

특히 배포 branch나 release tag 주변에서는 "누가 처음 변경했는가"보다 "누가 언제 이 변경을 배포 기준 히스토리에 반영했는가"가 더 중요할 수 있다.

그래서 feature branch에서는 rebase로 commit을 정리할 수 있지만, main이나 배포 branch에서는 히스토리 rewrite를 신중하게 다뤄야 한다.
