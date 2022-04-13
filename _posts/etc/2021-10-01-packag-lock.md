---
layout: post
title: "package-lock.json 에 대해 알아보기"
date: 2021-10-01
author: Seongjin
categories: etc
---

![Untitled](https://user-images.githubusercontent.com/52827441/163088139-0a5ee289-c1dd-443a-be43-995546d3c9e6.png)

그동안 NPM 의 dependency version 관리 방식과 `package-lock.json` 에 대해 궁금한 점이 많았지만 어렴풋이 개념만 알고 큰 문제를 마주한 적이 없어 깊게 파보지 않았다.

이번에 회사에서 사용중인 Open Source `kubeflow/katib` UI 코드의 Angular version 을 8에서 12로 업데이트하는 [contribution 작업](https://github.com/kubeflow/katib/issues/1590)을 하게 되었는데, dependency 들의 버전 호환과 관련된 작업들을 수행하며 다시 궁금한 점들이 생겨 여러 글과 문서를 찾아보게 되었다.

아쉽게도 내가 궁금했던 내용들이 NPM 의 문서에 명시되어 있지 않아, 다양한 블로그 글들을 참고해야 했고 이번 기회로 쉽지 않게 얻은 지식들을 정리해보기로 했다.
(대부분의 글들이 [semver versioning syntax](https://devhints.io/semver) 에 대한 설명을 하고 있지만, 'version ranges 정책이 왜 사용되는지', 'package-lock.json 이 왜 필요한지'에 대한 근본적인 해결책을 알려주는 글을 찾기는 힘들었다.)

### package-lock.json 이란

우리가 npm 을 사용해 설치하는 dependency tree 의 정확하고 상세한 정보를 저장하는 파일이며 npm 이 `node_modules` tree 나 `package.json` 을 수정하면 자동으로 생성된다.

`package-lock.json` 파일은 왜 필요할까?

`package.json` 에서는 패키지들의 버전정보를 `[version range](https://semver.npmjs.com/)` 를 사용해 저장하고 있다.

즉, "이 패키지는 2.2.3 버전을 사용한다." 라고 명시할 수도 있지만 많은 경우 "우리 app 은 이 패키지의 2.2.x 버전들을 사용할 수 있다." 라는 방식으로 명시하는 방식이다.

이러한 방식은 다음과 같은 상황에서 문제를 발생시킬 수 있다.

- npm 의 버전이 다를 경우, npm 이 `node_modules` tree 를 구성하는 알고리즘이 달라 다른 tree 가 생성될 수 있다.
- `version range` 를 따르는 패키지의 경우 패키지를 설치하는 시점에 따라 각각 다른 최신 버전을 설치하게 된다.

이러한 문제로 인해 개발자들은 서로 다른 환경에서 개발하게 될 위험이 높아지며, 이로 인해 예상치 못한 많은 문제들에 노출된다.

`package-lock.json` 은 npm 을 통해 설치된 package 의 version 을 specific 하게 명시하여 위의 문제를 해결하도록 도와준다.

![Untitled 1](https://user-images.githubusercontent.com/52827441/163088126-8229e36b-52a7-4ee6-bd3c-37e4f00a773a.png)

package.json

![Untitled 2](https://user-images.githubusercontent.com/52827441/163088131-2e42e928-361b-4027-b64d-dfb0266b5dd6.png)

package-lock.json

> 상단의 `package.json` 에서는 `react` 의 version 을 range 로 표기하고 있고 아래 사진인 `package-lock.json` 에서는 현재 node_modulse 에 설치된 `react` 가 정확히 어떤 버전인지 명시하고 있다.
> 

즉, 이 lockfile 은 Source Repository 에 Commit 을 하기 위한 용도로 만들어졌으며, 다음과 같은 용도로 사용된다.

- 협업, 배포, CI 등 모든 과정에서 동일한 dependency tree 를 구성할 수 있다.
- lockfile 에 `node_modules` tree 의 history 가 저장되어, `node_modules` 디렉토리를 따로 commit 할 필요가 없어진다.
- dependency tree 의 변경 사항에 대한 시각적인 diff 를 통해 가독성을 높일 수 있다.
- npm install 시 이전에 설치된 패키지의 반복되는 metadata 를 skip 할 수 있도록 하여 설치를 최적화 시킬 수 있다.

### package.json 은 왜 version ranges 정책을 따르는가?

위에서 언급한 것과 같이 NPM 의 version ranges 정책은 `npm install` 을 수행하는 시점에 따라 설치하는 Dependency 들의 버전이 다를 수 있다는 위험성이 있다.

물론 위에서 이에 대한 해결책인 `package-lock.json` 에 대해 알아보았지만, 만약 이 lockfile 이 없다면 npm 은 package 들의 versioning syntax 가 허용하는 범위의 최신 버전을 설치할 것이다. 

(또는 lockfile 이 있어도  `npm update` 를 입력하면 동일한 동작이 수행되기도 한다.)

**그럼 왜 애초에 이런 Risky 한 가능성을 열어두게 된 것일까?**

구글링을 통해 찾게된 한 [블로그](https://blog.greenkeeper.io/introduction-to-version-ranges-e0e8c6c85f0f)에 의하면,

version ranges 는 사용 중인 package 의 업데이트 될 버전을 의도적으로 허용하여 매번 최신 release 버전을 확인하고 `package.json` 을 업데이트하는 수고를 덜어주기 위한 기능이라고 한다. 

그리고 npm 이 내부적으로 다양한 기능을 수행하는데 필요하다고 하는데, 정확히는 모르지만 사용중인 package 의 peerDepedencies 와의 버전 호환과 관련이 있지 않을까 생각 된다.

NPM 은 `npm install` 시 default 로 caret(^) 을 version prefix 로 사용하여 설치를 한다.
`^` 는 semantic versioning ([semver](https://semver.org/lang/ko/)) 문법으로, 명시된 버전보다  `major.minor.patch` 중 minor 와 patch 의 상위 버전을 허용한다는 의미이다.

만약 project 에서 사용중인 dependency 의 버전이 민감하다고 여겨진다면
아래의 방법으로 prefix 없이 static 한 버전을 설치할 수 있다.

```bash
npm i --save-exact { package 명 }@{ version (default 는 latest) } 
```

또는 다음 설정으로 npm 의 default prefix 를 변경 할 수 있다.

```bash
npm config set save-prefix '원하는 prefix 또는 빈 문자열'
```

### package-lock.json 파일은 왜 갱신되는가?

이미 `package-lock.json` 이 존재하는 source 의 repository 를 clone 받아 `npm install` 을 할때, 종종 package-lock.json 파일의 version 이 갱신되어 git diff 로 추가되는 것을 볼 수 있다. 이런 경우 아래와 같은 이유일 수 있다.

- 오래된 lockfile 이 존재하는 경우
    
    npm version 7 부터 lockfile 에 저장되는 정보가 달라졌다. 좀 더 구체적으론 package 에 대한 전체 tree 정보를 저장해 패키지 설치 시  `package.json` 을 확인해야할 필요성을 낮추어 설치 성능을 향상시키기 위함이라고 한다.
    
    즉, 패키지 설치 과정에서 npm 이 version 6 이전의 npm 으로부터 생성된 `package-lock.json` 을 발견하면 자동으로 해당 파일을 업데이트하며, 이 업데이트는  `node_modules` tree 에 대한 누락된 데이터 또는 (lock 파일의 format 이 너무 오래되었거나 `node_modules` tree 가 비어있었을 경우) npm registry 에 대한 정보들을 저장한다.

    ![Untitled 3](https://user-images.githubusercontent.com/52827441/163088135-d994d16c-64e6-4438-bdd0-9218e3b26f3b.png)
    
    위의 상황이 발생한 예시
    

### Reference

- [https://junwoo45.github.io/2019-10-02-package-lock/](https://junwoo45.github.io/2019-10-02-package-lock/)
- [https://docs.npmjs.com/about-semantic-versioning](https://docs.npmjs.com/about-semantic-versioning)
- [https://docs.npmjs.com/cli/v7/configuring-npm/package-lock-json](https://docs.npmjs.com/cli/v7/configuring-npm/package-lock-json)
- [https://bytearcher.com/articles/semver-explained-why-theres-a-caret-in-my-package-json/](https://bytearcher.com/articles/semver-explained-why-theres-a-caret-in-my-package-json/)
