# Rebase 범위와 기준점 전략

`rebase`에서 가장 중요한 질문은 "어디로 옮길 것인가"보다 "어디부터 다시 적용할 것인가"다.

많은 실수는 rebase 대상 branch를 잘못 고르는 데서 생기는 것이 아니라, Git이 replay할 commit 범위를 잘못 잡게 만드는 데서 생긴다.

## Rebase는 Branch 전체가 아니라 Commit 범위를 Replay한다

아래 상태를 가정한다.

```text
A---B---C main
     \
      D---E---F old-age
```

여기서 `git rebase main`을 실행하면 Git은 `main..old-age` 범위의 commit을 새 base 위에 다시 만든다.

```text
A---B---C main
         \
          D'---E'---F' old-age
```

중요한 점은 `D`, `E`, `F`가 모두 replay 대상이라는 점이다. Git은 "이 branch에서 과거에 한 번 rebase했던 commit"이라는 이유만으로 `D`, `E`를 자동 제외하지 않는다.

Git이 보는 기준은 단순하다.

- 새 base에 없는 commit인가
- 현재 branch에는 있는 commit인가
- 그렇다면 replay 대상인가

따라서 "이미 rebase한 commit"이라는 표현은 사람의 작업 기억일 뿐, Git의 영구적인 상태 표시는 아니다.

## 이미 Rebase한 Commit이 다시 보이는 이유

이미 정리한 commit이 다시 rebase 대상처럼 보이는 대표적인 이유는 아래와 같다.

- 기준 branch에 그 commit hash가 아직 들어가지 않았다
- squash merge나 cherry-pick으로 반영되어 내용은 비슷하지만 hash가 다르다
- stacked branch 구조에서 하위 branch commit까지 함께 포함되어 있다
- `git rebase <target>`만 사용해서 제외해야 할 upstream boundary를 명시하지 않았다
- 오래 열린 branch에 후속 commit을 계속 쌓으면서 branch의 책임 범위가 커졌다

특히 squash merge 이후에는 주의해야 한다. main에는 같은 변경 내용이 들어갔더라도 원래 feature branch의 commit hash는 없다. 이 상태에서 기존 feature branch를 계속 재사용하면 Git 입장에서는 기존 commit들이 아직 main에 없는 commit처럼 보일 수 있다.

## 다시 Replay되는 것이 항상 문제는 아니다

이미 rebase했던 commit이 다시 replay된다고 해서 항상 잘못된 것은 아니다.

예를 들어 `D'`, `E'`가 아직 main에 들어가지 않았고 main만 앞으로 이동했다면, `D'`, `E'`, `F`를 모두 새 main 위로 다시 만드는 것이 정상이다.

```text
A---B---C---G---H main
         \
          D'---E'---F old-age
```

이 상태에서 `old-age`를 최신 main 위로 올리려면 결과는 아래처럼 되어야 한다.

```text
A---B---C---G---H main
                 \
                  D''---E''---F' old-age
```

`D'`, `E'`가 다시 만들어지는 이유는 비효율 때문이 아니라 parent가 `C`에서 `H`로 바뀌기 때문이다. parent가 바뀌면 commit object도 새로 만들어져야 한다.

문제가 되는 경우는 `D'`, `E'`가 이미 main에 반영됐다고 판단했는데도 branch에 계속 남아 있어서 다시 replay되는 상황이다. 이때는 rebase를 반복할 문제가 아니라 branch를 새로 만들거나 replay 범위를 잘라야 하는 문제다.

## 먼저 Replay 범위를 확인한다

rebase하기 전에 Git이 어떤 commit을 replay할지 먼저 확인해야 한다.

```bash
git log --oneline <new-base>..HEAD
```

예를 들어 main 위로 rebase하려면 아래 명령으로 replay 후보를 본다.

```bash
git log --oneline origin/main..HEAD
```

여기에 이미 통합됐다고 생각한 commit이 보인다면, 바로 rebase하지 말고 기준점을 다시 잡아야 한다.

패치 기준으로 target에 이미 들어간 변경인지 확인하려면 아래 명령도 유용하다.

```bash
git cherry -v origin/main HEAD
```

이 명령은 commit hash가 달라도 patch-id 기준으로 비슷한 변경이 target에 있는지 판단하는 데 도움을 준다.

- `+`는 target에 같은 patch가 없을 가능성이 높다는 뜻이다
- `-`는 target에 같은 patch가 이미 있을 가능성이 높다는 뜻이다

다만 실제 운영 판단은 `git log`, diff, PR 병합 방식까지 함께 봐야 한다.

## 기준점이 두 개라는 사실

rebase를 정확히 이해하려면 기준점이 두 개라는 점을 분리해야 한다.

- `new base`
  commit을 새로 올려놓을 목적지
- `upstream boundary`
  replay에서 제외할 과거 경계

단순한 rebase에서는 이 둘이 같은 값처럼 보인다.

```bash
git rebase origin/main
```

하지만 이 명령은 실제로 아래 의미에 가깝다.

```text
origin/main에 없는 현재 branch commit들을 origin/main 위에 다시 만든다
```

반면 "이미 정리된 commit은 제외하고 새 commit만 옮기고 싶다"면 목적지와 제외 경계를 따로 지정해야 한다.

```bash
git rebase --onto <new-base> <upstream-boundary> <branch>
```

의미는 아래와 같다.

```text
<branch>에서 <upstream-boundary> 이후 commit만 골라 <new-base> 위에 다시 만든다
```

## 이미 통합된 Commit 뒤의 새 Commit만 옮기기

예를 들어 `old-age` branch에서 `D'`, `E'`는 이미 main에 squash merge나 별도 방식으로 반영됐고, 이후 `F`만 새로 추가했다고 가정한다.

```text
A---B---C---S main
         \
          D'---E'---F old-age
```

여기서 `S`는 `D'`, `E'`를 squash한 commit이다. main에는 `D'`, `E'` hash가 없기 때문에 단순히 `git rebase main`을 하면 Git이 `D'`, `E'`, `F`를 모두 후보로 볼 수 있다.

이때 정말 필요한 것이 `F`뿐이라면 선택지는 두 가지다.

첫 번째는 새 branch를 main에서 만들고 필요한 commit만 cherry-pick하는 방식이다.

```bash
git switch -c old-age-followup origin/main
git cherry-pick <F>
```

두 번째는 `--onto`로 replay 범위를 직접 자르는 방식이다.

```bash
git rebase --onto origin/main <E'> old-age
```

이 명령은 `old-age`에서 `<E'>` 이후 commit만 `origin/main` 위로 옮긴다. 즉, `F`만 replay한다.

다만 이 방식은 `<E'>` 이전 변경이 새 base에 이미 들어갔다는 확신이 있을 때만 안전하다. 들어가지 않은 변경을 제외하면 필요한 코드가 사라진다.

## Stacked Branch에서는 하위 Branch까지 같이 딸려올 수 있다

stacked branch 구조에서는 이 문제가 더 자주 생긴다.

```text
A---B---C main
         \
          D---E base-topic
               \
                F---G old-age
```

`old-age`를 main 위로 단순 rebase하면 `D`, `E`, `F`, `G`가 모두 replay 대상이 된다. `old-age`는 `base-topic`의 commit을 포함하고 있기 때문이다.

`F`, `G`만 다른 base로 옮기고 싶다면 하위 branch의 끝을 upstream boundary로 지정한다.

```bash
git rebase --onto origin/main base-topic old-age
```

의미는 아래와 같다.

```text
old-age에서 base-topic 이후 commit만 origin/main 위로 옮긴다
```

이것이 `git rebase <target>`과 `git rebase --onto <new-base> <upstream> <branch>`의 핵심 차이다.

## Rebase 전에 Commit을 정리해야 하는 이유

commit history를 잘 정리한 뒤 rebase해야 하는 이유는 단순히 예쁘게 보이기 위해서가 아니다.

rebase는 commit 단위로 replay된다. 따라서 commit이 지저분하면 아래 비용이 커진다.

- 충돌이 여러 commit에서 반복된다
- 어떤 commit이 이미 통합된 변경인지 판단하기 어렵다
- `--onto`의 upstream boundary를 잡기 어렵다
- squash merge 이후 중복 변경을 구분하기 어렵다
- 리뷰어가 branch의 실제 의도를 파악하기 어렵다

작업 중에는 commit이 지저분해도 된다. 하지만 main이나 배포 branch에 올리기 전에는 `rebase -i`로 먼저 의미 있는 단위로 정리하는 편이 좋다.

```bash
git rebase -i origin/main
```

다만 이 명령을 실행하기 전에도 `origin/main..HEAD` 범위가 맞는지 먼저 확인해야 한다. stacked branch나 squash merge 이후 후속 작업이라면 `origin/main`이 아니라 실제 upstream boundary를 기준으로 interactive rebase해야 할 수 있다.

## 운영 전략

반복 rebase를 안전하게 하려면 아래 기준을 지킨다.

- rebase 전에 `git log <target>..HEAD`로 replay 범위를 본다
- 이미 통합된 작업에 후속 작업을 계속 쌓지 말고 새 branch를 만든다
- squash merge한 branch는 보통 재사용하지 않는다
- stacked branch는 아래 branch부터 위 branch 순서로 rebase한다
- 일부 commit만 옮길 때는 `--onto`로 upstream boundary를 명시한다
- rebase 전에는 임시 branch나 tag로 원래 위치를 남겨 둔다
- 공용 branch를 rewrite했다면 push는 `--force-with-lease`를 사용한다

임시 기준점을 남기는 예시는 아래와 같다.

```bash
git branch backup/old-age-before-rebase HEAD
```

이렇게 해두면 rebase가 꼬였을 때 원래 branch 상태를 다시 비교하거나 복구할 수 있다.

## 결론

rebase 전략의 핵심은 "최신 main 위로 올린다"가 아니다.

정확한 핵심은 아래 질문에 답하는 것이다.

- 어떤 commit들이 아직 통합되지 않았는가
- 어떤 commit부터 replay해야 하는가
- 어디를 새 parent로 삼을 것인가
- 이미 squash, cherry-pick, rebase된 commit을 다시 포함하고 있지는 않은가

이 질문에 답하지 않은 채 `git rebase <target>`만 반복하면 이미 정리한 commit까지 다시 움직이는 것처럼 보일 수 있다. rebase는 강력하지만, 기준점을 명시하지 않으면 너무 넓은 범위를 다시 쓸 수 있다.
