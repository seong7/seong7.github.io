---
layout: post
title: "Open Source Project 에 PR 날리는 방법"
date: 2021-06-26
categories: etc
---

![Untitled](https://user-images.githubusercontent.com/52827441/163081744-f9fa0b68-3176-477a-9d01-547cfa06e364.png)


일반적으로 Github 에서 다른 사람의 Repo 에 PR 을 날리기 위해서는 Repository owner 로부터 permission 을 받아야 한다.

하지만, Open source 의 수 많은 contributer 들이 모두 permission 을 받은 것은 아니다.

어떻게 PR 을 날릴 수 있는지 알아보자.

### 1. Fork Repository

Original Repo 를 본인의 Repo 로 Fork 한다.

** 만약 Original Repo 를 먼저 clone 해서 작업해버렸다면 5**번**으로 넘어가면 된다.

### 2. Clone and Code

Fork 한 Repository 를 clone 하여 작업한다.

### 3. Commit and Push

작업한 내용을 commit 한 후,
Fork 한 Repository 에 push 한다.

### 4. Push 된 브랜치로 PR 생성

Original Repo ← Forked Repo 

![Untitled 1](https://user-images.githubusercontent.com/52827441/163081724-55e6331b-b9a5-48da-9a9d-d0b13a9517fb.png)

### 5. 만약, Original Repo 를 clone 해서 작업해버렸다면 !?

- 5-1. Fork Repository

- 5-2. local git repo 에 Fork 한 remote repo 를 설정해준다.
    
    ```bash
    $ git remote add {remote repo 의 alias ('origin' 제외)} {Forked Repo 의 clone url}
    
    # 모든 remote repository 들을 확인한다.
    $ git remote -v
    
    # Fork 한 remote repository 에 push
    $ git push {Forked - remote repository 의 alias} {local branch}
    ```
    
    실행 화면 :

    ![Untitled 2](https://user-images.githubusercontent.com/52827441/163081737-672e1f76-938c-4551-9243-0ef0ec3e22a3.png)




- 5-3. 이제 3번처럼 PR 을 생성할 수 있다.
