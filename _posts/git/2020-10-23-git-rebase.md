---
layout: post
title: "Git Rebase 이해하기"
date: 2020-10-23
categories: Git_&_Github
---

# Git Rebase

`git rebase` 는 초보 개발자들에게 피하고 싶은 git command 로 여겨지지만, 잘 사용하면 개발팀에 매우 유용할 수 있다.

이 글에서는 `git rebase` 와 `git merge` 를 비교하며 Git workflow 에서 rebasing 을 사용할만한 모든 경우들을 설명해보려한다.

## 개념

가장 먼저 `git rebase` 에 대해 이해해야할 것은 `git merge` 와 **동일한 문제**를 해결한다는 것이다.  
두 command 모두 한 branch 에서의 change 를 다른 branch 에 통합하는 목적을 갖고 있다. 하지만 이를 매우 다른 방법으로 수행한다.

**새로 update 된 master 브랜치의 commit 들을 내가 작업 중인 feature 브랜치에 통합시킨다고 가정해보자.**

### Merge 할 경우

```sh
git checkout feature
git merge master

또는 한줄로
git merge feature master
```

merging 은 feature 브랜치에 새로운 **merge commit** 을 생성한다. 두 브랜치의 history 를 합친 commit 이 발생하는 것이다.

<a href="https://user-images.githubusercontent.com/52827441/96963075-f6f1d200-1542-11eb-80ba-1547653f925f.png" data-lightbox="merge commit" data-title="merge commit">
<img src="https://user-images.githubusercontent.com/52827441/96963075-f6f1d200-1542-11eb-80ba-1547653f925f.png" alt="merge commit" style="height:350px; width:auto;">
</a>

Merging 은 안전하다. 기존 브랜치의 내용을 유지하고 새로운 commit 을 생성하기 때문이다.

하지만, <u>만약 master 브랜치가 매우 active 하게 update 되고 있다면?</u>  
<u>그에 따라 나의 feature 브랜치와 계속해서 merge 해야한다면?</u>

**내 feature 브랜치의 history 는 merge commit 으로 인해 꽤나 어지럽혀질 것이다.**

<br/>

### Rebase 할 경우

Merge 대신 다음 명령어로 Rebase 작업을 할 수 있다.

```sh
git checkout feature
git rebase master
```

rebasing 은 feature 브랜치를 master 브랜치의 끝에서부터 시작하도록 변경한다.  
다시 말해, **feature 브랜치의 기존 commit 들에 대응하는 새로운 commit 들을 하나씩 생성하여 project 의 history 를 *재작성(re-write)*한다.**

<a href="https://user-images.githubusercontent.com/52827441/96964924-44bc0980-1546-11eb-8739-103f99c503f3.png" data-lightbox="rebase" data-title="rebase">
<img src="https://user-images.githubusercontent.com/52827441/96964924-44bc0980-1546-11eb-8739-103f99c503f3.png" alt="rebase" style="height:350px; width:auto;">
</a>

rebasing 의 가장 큰 장점은 훨씬 깔끔한 project history 이다. 위의 그림에서 볼 수 있듯 **완벽한 직선형태의 history** 를 남길 수 있다. 이는 `git log`, `git bisect` 또는 `gitk` 등의 명령어를 사용한 **project navigating 을 훨씬 쉽게 해준다.**

<br/>

하지만, 이렇게 깨끗한 commit history 는 **두 가지 trade-off** 를 가진다.

**1. Safety (안전성)**  
만약 **rebasing 의 황금룰**([Golden Rule of Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing))을 따르지 않는다면, history 재작성 기능은 협업 workflow 의 **잠재적 위험요소**가 될 것이다. <sub>(아래에서 설명하고 있음)</sub>

**2. Traceability (추적 가능성)**  
1 번 보다는 덜 중요하지만, commit history 에서 merge commit 이 없어지므로 어디서부터 새로운 feature 브랜치가 합쳐졌는지 알 수 없다.

<br />

## Interactive Rebasing

Interactive Rebasing 은 Rebase 를 실행하기 전, **내가 작업 중인 feature 브랜치의 commit history 를 조작**할 수 있게 해준다.

```sh
git rebase -i master
```

위의 명령어를 실행하면 아래와 같이 editor 가 나타난다.

<a href="https://user-images.githubusercontent.com/52827441/97015318-6341f500-1586-11eb-8006-e01ac910caa0.png" data-lightbox="interactive rebase" data-title="rebase">
<img src="https://user-images.githubusercontent.com/52827441/97015318-6341f500-1586-11eb-8006-e01ac910caa0.png" alt="interactive rebase" style="height:450px; width:auto;">
</a>

위쪽은 **commit 기록들을 조작**할 수 있는 부분이고 아래는 **설명** 부분이다.

각각의 commit 들의 앞에 붙은 command 에 따라 조작이 이루어지며 각 command 는 설명 부분에 나열되어 있다.

- pick : 해당 commit 그대로 유지
- reword : 해당 commit 의 message 변경
- fixup : 해당 commit 을 없애고 코드 변경 사항은 바로 윗 commit 에 포함 시킴  
  (모든 command 를 숙지하진 못했고 사용하면서 익혀가야겠다.)

**Interative Rebasing** 은 보통 불필요한 commit history 를 정리하기 위해 사용한다. commit 기록을 완전히 제어할 수 있으므로 자동 rebasing 보다 powerful 하다.

물론 git merge 에서는 할 수 없는 기능이다.

<br />

## Golden Rule of Rebasing

Rebasing 을 어느정도 이해했다면, 가장 중요한 것은 **언제 git rebase 를 사용하면 안되는지**이다.

<mark>git rebase 는 public 브랜치에서는 절대로 사용하면 안된다.</mark>

특히 **master** 나 **dev** 브랜치가 public 브랜치에 해당하는데, 이처럼 다른 팀원과 공유하는 브랜치를 rebase 할 경우, 해당 브랜치는 내 로컬에서만 **전혀 다른 history** 를 갖게 된다.

이를 다시 다른 팀원들의 동일 브랜치와 합치려면 `git merge` 를 통해 새로운 merge commit 을 생성하는 것이 불가피하다. 그뿐만 아니라 history 상에 **동일한 code 변경 내용을 가진 history 가 중복**으로 존재하게 된다. 한마디로 매우 혼란스러운 상황이다.

그러니 **`git rebase` 를 실행하기 전, 해당 브랜치가 다른 사람과 공유되고 있지 않은지 반드시 재확인해야한다.**

(branch 정책을 반드시 exclusive 하게 정해야 할듯)

만약 다른 사람과 공유되고 있는 브랜치가 맞다면, `git rebase` 가 아니라 **비파괴적인 (non-destructive) 방법**을 생각해봐야 한다. (e.g., `git revert`)

(이런 경우가 생기면 `git revert` 에 대한 학습 또한 필요할 것 같다.)

<br />

## Local Cleanup

작업 중인 feature 브랜치의 commit 몇 개만의 history 를 수정하려면, `git rebase` 의 base 를 master 브랜치의 끝이 아니라 feature 브랜치의 이전 commit 중 하나로 정해주면 된다.

```sh
git rebase -i HEAD~3
```

위의 명령어로 HEAD 에서 3 단계 전의 commit 을 base 로 잡고 그 위의 commit 부터 history 를 re-write 할 수 있다.

<a href="https://user-images.githubusercontent.com/52827441/97101007-effec700-16dc-11eb-81a3-47cec048ae35.png" data-lightbox="git rebase -i HEAD~3" data-title="git rebase -i HEAD~3">
<img src="https://user-images.githubusercontent.com/52827441/97101007-effec700-16dc-11eb-81a3-47cec048ae35.png" alt="git rebase -i HEAD~3" style="height:350px; width:auto;">
</a>

위의 그림과 같이 Master 브랜치의 끝으로 Feature 브랜치를 이동시키지 않고 local (Feature) 에서만 history 를 정리할 수 있으므로 **Local Cleanup** 이라고 한다.

Feature 브랜치의 전체를 Local 에서만 정리하고 싶다면 아래의 과정을 따른다.  
(Master 브랜치의 끝으로 이동시키지 않을 때)

아래 명령어를 입력하면 해당 브랜치의 **original base** <sub>(위 그림에 표시됨)</sub> 를 반환해준다.

```sh
git merge-base feature master
```

그리고 아래 명령어를 입력한다.

```sh
git rebase -i <위 명령어에서 반환된 original base>
```

<br />

## git pull 로 rebase 하기

<a href="https://user-images.githubusercontent.com/52827441/97101179-8384c780-16de-11eb-90cb-a7c0ad5371d8.png" data-lightbox="git pull rebase - 1" data-title="git pull rebase - 1">
<img src="https://user-images.githubusercontent.com/52827441/97101179-8384c780-16de-11eb-90cb-a7c0ad5371d8.png" alt="git pull rebase - 1" style="height:350px; width:auto;">
</a>

위와 같이 John 이라는 팀원의 Feature 브랜치와 나의 Feature 브랜치가 각각 작업되고 있고 내 브랜치에 John 의 수정사항을 합치고 싶다면 아래를 따른다.

```sh
git pull --rebase <remote repo> Feature
```

그 결과는 아래와 같다. `git merge` 보다 더 직관적으로 'John 이 이미 작업한 내용에 내 작업물을 추가해' 라는 명령이 적용되었다고 볼 수 있다.

<a href="https://user-images.githubusercontent.com/52827441/97101468-0444c300-16e1-11eb-8c76-6267258c39aa.png" data-lightbox="git pull rebase - 2" data-title="git pull rebase - 2">
<img src="https://user-images.githubusercontent.com/52827441/97101468-0444c300-16e1-11eb-8c76-6267258c39aa.png" alt="git pull rebase - 2" style="height:350px; width:auto;">
</a>

<br />

## Feature 브랜치 합치기

이미 언급한대로 Master 브랜치의 history 를 수정하면 안되므로, 결국에는 `git merge` 를 통해 Feature 브랜치를 Master 브랜치에 합친다.

하지만, Merge 전 Local Cleanup (rebasing) 을 하면 Merge 가 **fast-fowarded** 로 이루어지고 또한 **깔끔한 직선 history** 를 결과로 남길 수 있다.

아직 `git rebase` 사용이 미숙하다면, temporary 브랜치를 생성해 rebase 를 작업할 수 있다. 실수로 Feature 브랜치의 history 를 잘못 조작하더라도 original Feature 브랜치로부터 새로운 temporary 브랜치를 생성해 다시 rebase 할 수 있다.

```sh
git checkout -b temporary-branch

git rebase -i master
# [Clean up the history (history 정리)]

git checkout master

git merge temporary-branch
```

<br />

### Reference

- [BitBucket document - Git Rebase](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

<br />

## 남아 있는 의문점

- 하나의 branch 를 pull request 날리고 나면 해당 branch 에는 더이상 `git rebase` 하면 안되는지? (보통 pull request 날린 후 master 에 Merge 하고 나서도 bug fix 나 수정 사항을 작업해 동일한 branch 로 PR 날리는 것으로 알고 있는데..)

- GitHub 에서 Feat 브랜치 PR 을 Dev 나 Master 에 합치는 것은 여전히 `git merge` 로 이루어 지는 것이 맞는지?
