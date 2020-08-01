---
layout: post
title: "Create React App - eslint-watch 사용하기"
date: 2020-07-30
author: Jason
categories: react
published: true
---

## 개요

eslint 를 command line 에서 사용할 때 watch 모드로 실행하고 싶었다.  
[eslint-watch](https://www.npmjs.com/package/eslint-watch) 를 알게 되었고
문서에 설명된 방법에 따라 `yarn add -D eslint eslint-watch` 를 실행했는데...  
**문제** 가 발생했다.

Create React App 에는 이미 eslint 가 내장되어 있는데 내가 eslint 를 다른 버전으로 또 설치해버린 것이다. 그에 대한 문제 해결법은 아래에 정리해 두었다.

## 문제 발생

<a href="https://user-images.githubusercontent.com/52827441/88914505-aea57980-d29d-11ea-8c7d-4682b4f4f67c.png" data-lightbox="eslint error 1" data-title="eslint error 1">
<img src="https://user-images.githubusercontent.com/52827441/88914505-aea57980-d29d-11ea-8c7d-4682b4f4f67c.png" alt="eslint error 1"/>
</a>

몇몇 Package 는 Create React App 에서 자동으로 설치하므로 따로 설치할 경우 위와 같은 에러가 뜬다.

(정확히는 Error 는 아니고 서로 다른 버전의 dependency 중복에 의한 **hard-to-debug issue** 가 발생할 수 있다는 경고이다.)

<br>

## 해결법

전체적으로 위의 사진의 지시사항을 따른다. (CRA 에서 감사하게도 친절히 설명해두었다.)

2 번의 `node_modules` 삭제 방법은 아래와 같다.

<a href="https://user-images.githubusercontent.com/52827441/88914997-7488a780-d29e-11ea-85e2-7cc6ea182b6f.png" data-lightbox="eslint error 2" data-title="eslint error 2">
<img src="https://user-images.githubusercontent.com/52827441/88914997-7488a780-d29e-11ea-85e2-7cc6ea182b6f.png" alt="eslint error 2"/>
</a>

내 컴퓨터가 너무 느려서.. yarn install 은 지옥과 같다..ㅠ

<br>

## eslint-watch 실행하기

eslint 가 중복 설치된 것은 해결했고 이제 실행을 한다. (실행 script 는 문서를 참조했다.)  
그런데 watch 모드가 작동하지 않고 내가 수동으로 lint 를 매번 실행시켜줘야했다...

다시 한번 [eslint-watch 문서](https://www.npmjs.com/package/eslint-watch#requirements) 를 살펴보니 `"eslint": ">=7 <8.0.0"` 버전에서 사용 가능하다고 한다. 내 프로젝트는 `6.6.0` 버전에 의존하고 있다.

어떻게 eslint 버전을 업그레이드 할 수 있을까?

## CRA eject

eject 를 해줬다.  
솔직히 맞는 방법인지 모르겠다. 패키지들이 CRA 에 내장된 상태에서 eslint 의 version 을 업그레이드 하는 방법을 몰랐기 때문에 eject 를 하기로 했다.

eject 하는 방법은 쉽다. [[설명 링크](https://helloinyong.tistory.com/174)]

그 결과 내장되어 있던 패키지들과 환경 설정 코드들이 정체를 드러냈다.

> 혹시 eject 를 **undo** 하고 싶다면 **[링크 참조](https://stackoverflow.com/questions/51454729/undo-npm-run-eject-in-react)**

> devDependency 에 기존에 설치되어 있던 패키지들이 eject 후에 dependency 로 추가되어 기존 패키지들을 devDependency 에서 삭제해줬다.
>
> - eslint-plugin-react-hooks
> - eslint-plugin-react
> - eslint-plugin-jsx-a11y
> - eslint-plugin-import
>
> (devDependency 만 삭제는 안돼서 `yarn remove {package 명}` 로 둘 다 삭제 후 다시 dependency 로 add 해줬다.)

## eslint version upgrade

```
yarn upgrade eslint@7.5.0
```

이제 위 명령어로 eslint 만 version 을 upgrade 해줬다.

> 여기서 만약 오류가 발생한다면, 실행중이던 dev-server 를 종료하고 해보시길 추천한다.

## 결과

eslint-watch 가 코드의 변화를 감지하고 자동으로 re-run 된다.
