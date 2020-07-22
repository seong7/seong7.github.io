---
layout: post
title: "Github 블로그 (by Jekyll) 만들기 - 자료 모음"
date: 2020-07-14
author: Jason
categories: etc
---

# Jekyll 로 Github 블로그 만들기

꼬박 4 일에 걸쳐 Github 블로그를 릴리즈할 수 있었다.

앞으로 추가해야할 부분도 많지만 이쯤에서 지금까지 도움되었던 자료들을 정리해보려 한다.

혹시나 Github 블로그 만들기에 도전하는 분들이 있다면, 이 글을 통해 나보다 하루라도 덜 걸려 목표를 달성하시면 좋겠다. 🙂

<br>

### 1. Github Pages 배포하기

Github Pages 는 static 웹사이트를 호스팅 해주는 Github 의 서비스이다. Github repository 내 HTML, CSS, JavaScript 를 가져와 자동으로 내부 build process 를 거친 후 `github.io` 또는 custom 도메인으로 배포한다.

> static 사이트만 지원하므로 server-side 언어인 PHP, Ruby, Python 등은 지원하지 않는다.

우선 Github 개인 사용자의 사용 가능 정보는 아래와 같다.

| Github product 종류 | public repository | private repository |
| ------------------: | ----------------: | -----------------: |
|         Github Free |                 O |                  X |
|          GitHub Pro |                 O |                  O |

사용법은 간단하다.

`username`.github.io 을 이름으로 repository 를 생성하면 된다.

간단히 index.html 파일을 생성해 push 하면 약 30초 후 해당 파일이 https://`username`.github.io 링크에서 열리는 것을 확인할 수 있다.

<a href="https://user-images.githubusercontent.com/52827441/87873350-e1c54e80-c9fb-11ea-95a9-ae68e0e23967.png" data-lightbox="github1-large" data-title="github repository 만들기 화면">
<img src="https://user-images.githubusercontent.com/52827441/87873350-e1c54e80-c9fb-11ea-95a9-ae68e0e23967.png" alt="github1"/>
</a>

내가 알기로는 master branch 만 배포가 가능하고 다른 branch 에서 배포를 테스트할 수는 없는 것으로 알고 있다.

<br>

### 2. Ruby 및 Jekyll 설치하기

Github Pages 서비스는 **[공식문서](https://docs.github.com/en/github/working-with-github-pages/about-github-pages#static-site-generators)** 에서 'static file 을 통한 직접적인 사이트 build' 또는 '**Static site generator**<sub> (정적 사이트 생성기)</sub> 를 사용한 사이트 build' 가 가능하다고 설명하고 있다.
그리고 **Static site generator 중 Jekyll 을 추천**하고 있다.

이런 이유로 내가 택한 방법 또한 Jekyll 이다.

Jekyll 은 Ruby 로 작성되었으며 평범한 Ruby 의 Gem 이다.

> 짧은 설명을 보고 내가 이해한 Gem 은 Node.js 의 NPM / Yarn 패키지와 유사한 개념이다.  
> (혹시 제가 잘 못 이해하고 있다면 댓글로 정정 부탁드립니다 !)

설치 방법은 **[Jekyll 공식 문서 한국어 버전](https://jekyllrb-ko.github.io/docs/installation/)** 을 보고 따라하는 것을 추천한다.  
설치 외에도 루비의 기초 등 다양한 설명들이 큰 도움이 된다.

<br>

### 3. Jekyll 로 블로그 생성하고 local 에서 열기

Jekyll 에서는 **default theme** 을 제공하고 있다.  
즉, Jekyll 로 처음 웹사이트를 생성하면 평범하고 깔끔한 스타일로 사이트가 생성되어 있음을 확인할 수 있다.

사이트 생성 방법은 **[Jekyll 공식문서의 빠른시작 - 절차 부분](https://jekyllrb-ko.github.io/docs/#instructions)** 을 참조하면 된다.

문서의 설명대로 사이트 생성 후 `bundle exec jekyll serve` 를 통해 http://localhost:4000 에 serve 할 수 있는데, 나는 아래 옵션들을 추가해서 사용하고 있다.

- `--watch` : 파일 변화를 감지해 자동 re-build 해준다.
- `--incremental` : 증분 재생성 (변화된 페이지만 re-build) 해 build 속도를 가속 시킨다.  
  (수정 : --incremental 은 완전한 build 를 해내지 못할 위험성이 있다는 글을 읽어 더 이상 사용하지 않음, 실제로도 변경 내용이 잘 적용되지 않는 문제점 있었음)

```
bundle exec jekyll serve --watch --incremental
```

> 개인적으로 **`jekyll 블로그명`** 명령어로 Jekyll default 사이트를 생성하는 단계를 건너 뛰고 4번으로 갔다. (이유는 4번에서 설명)

<br>

### 4. Jekyll theme 고르고 적용하기

Jekyll 의 default theme 을 사용하지 않을 거라면,  
그리고 직접 처음부터 사이트를 styling 하지 않을 거라면,  
다음의 과정을 통해 이미 만들어진 다양한 Jekyll theme 을 적용할 수 있다.

수 많은 custom theme 들이 open source 로 만들어져있다. (Jekyll theme 을 전문적으로 만들어 유료로 판매하는 분들도 있는 것 같다.)

개인적으로 jekyll theme 모음 사이트는 **[여기](http://jekyllthemes.org/)** 가 가장 괜찮아 보였다.  
하지만 나는 선별된 theme 들만 보고 싶어 google 에 Jekyll theme 추천을 검색했고 **[Aiden 님이 블로그](https://isme2n.github.io/devlog/2017/03/09/Blog-Jekyll-theme/)** 에 추천해두신 theme 을 적용했다.

Theme 이름은 **[Centrarium](https://github.com/bencentra/centrarium)** 이다.

#### 내가 Theme 을 고른 기준은 아래와 같았다.

1. **내가 구상하는 blog 구조와 일치하는가?**

   이 부분이 가장 중요하다고 생각한다. `5번` 과 같이 jekyll theme 을 커스터마이징 할 수 있지만, 사이트 구조 자체를 변경하는 것은 정말 쉽지 않아 보였다.

   내가 고른 theme 은 사이트 구조가 내 블로그의 모티브인 **[초보 몽키님의 블로그](https://wayhome25.github.io/)** 와 가장 유사해 선택했다.

2. **document 를 잘 정리해둔 theme 인가?**

   이 부분은 특히 Installation 과 Customizing 에 중요하다.
   내가 `4번` 에서 `Jekyll default 사이트 생성 과정` 을 건너뛸 수 있었던 이유는 내가 고른 Centrarium Theme 의 Github repo **[README](https://github.com/bencentra/centrarium)** 가 잘 작성되어 있었기 때문이다. 문서의 Installation 부분에 따르면 Jekyll 을 처음 시작한다면 해당 Theme 의 파일들을 다운 받아 폴더에 저장한 후 dependency 들을 설치하기만 하면 된다고 설명되어 있다.

   그 외에도 문서를 참조해야할 내용이 많다.

3. **미리보기 사이트를 지원하는가?**

   Demo 사이트를 지원하지 않으면 일일히 Theme 을 설치 받아 확인해야하는데 그러고 싶진 않았다.

4. **원하는 기능을 지원하는가?**

   [Disqus](https://help.disqus.com/en/) 나 [utterances](https://utteranc.es/) 등 댓글 기능을 지원하는가, 또는 Gooogle Analytics 를 지원하는가?

   이런 기능들은 문서를 찾아보고 직접 적용할 수도 있지만 편의에 따라 결정하면 된다.

   참고로 내가 고른 Centrarium 은 Disqus 와 Google Analytics 가 이미 포함된 theme 이다. 간단한 설정으로 사용할 수 있었다.

<br>

### 5. Jekyll theme 커스터마이징하기

Jekyll 은 shopify 에서 만든 **[Liquid](https://shopify.github.io/liquid/)** 라는 언어를 사용한다.

Jekyll 로 생성된 사이트의 기본적인 디렉터리 구조와 Liquid 문법은 **[이지혜님의 블로그 글](http://jihyeleee.com/blog/third-designer-can-make-jekyll-blog/)** 을 참조로 간략하게 이해했다.

그 후 직접 Source 를 뜯어보며 작업했다. Source 를 이해하는데 팁은 `_config.yml` 과 `.md` 파일들을 수정함에 따라 `_site` 폴더 내 디렉터리 구조나 `.html` 파일들의 코드가 어떻게 생성되는지 보며 작업하면 이해해나갈 수 있을 것이다. 그리고 <u>역시나 stackoverflow / 구글링이 최고다 !</u>

**종종 변경한 부분이 잘 적용되지 않을 때가 있는데 내가 사용하는 방법은 아래와 같다.**

1.  **\_site 폴더를 삭제한다. (server 재시작 필요)**  
    \_site 폴더는 jekyll 이 markdown, \_config.yml, style sheet 등의 파일들을 읽고 실제 render 할 website 를 build 하여 저장하는 폴더이다. re-build 할 때 새로 생성되므로 삭제하여 재생성 시킨다.
2.  **browser 개발자 도구 활용**  
    흔히 browser 에 저장된 cache 로 인해 변경사항이 적용되지 않을 때가 있다. 이럴 때는 개발자 도구의 Network 에서 `disable cache` 를 선택하고 개발자 도구를 켜둔채로 reload 한다.

> **\_config.yml 파일을 수정한 경우**  
> 수정사항을 bundler 가 자동으로 감지하지 못하므로 server 를 반드시 재시작해줘야 반영된다.

<br>

### 6. 내 블로그가 구글에서 검색되도록 하기

Jekyll 로 생성한 블로그는 default 로 Google 에서 검색되지 않는다.

검색되도록 하기 위해 **[초보 몽키님의 블로그 글](https://wayhome25.github.io/etc/2017/02/20/google-search-sitemap-jekyll/)** 을 따르기를 추천한다.

적용 후 시간이 조금 흐르면 검색이 가능해진다.

<br>

### 기타 도움되는 사이트 모음

- **logo 및 favicon 만들기**

  - [무료 logo 생성 사이트](https://www.freelogodesign.org/)
  - [무료 favicon 파일 생성 사이트](https://www.favicon-generator.org/)

- **캐릭터 생성** : Jepeto 앱 사용

- **유니코드 이모티콘 테이블** : [웹사이트](https://unicode-table.com/en/)

<br>

### 앞으로 적용할 부분

아래 부분은 나도 아직 적용 전이라 자료에 확신은 없다.  
메모 용으로 써두었다.

검색 기능 적용 :  
http://labs.brandi.co.kr/2019/04/15/chunbs.html  
https://github.com/christian-fei/Simple-Jekyll-Search

<br>

## 출처 모음

1. [Github 공식 문서 - Github pages ](https://docs.github.com/en/github/working-with-github-pages/about-github-pages)

2. [Jekyll 공식 문서 한국어 버전](https://jekyllrb-ko.github.io/docs/)

3. 내가 고른 Jekyll theme 을 추천해주신 [Aiden 님의 블로그](http://jekyllthemes.org/)

4. 내 블로그의 모티브인 [초보몽키 님의 블로그](https://wayhome25.github.io/)

5. 내가 고른 Jekyll theme ['Centrarium' 의 Gihub Repo](https://github.com/bencentra/centrarium)

6. Jekyll 사이트의 디렉터리 구조를 설명하신 [이지혜님의 블로그](http://jihyeleee.com/blog/third-designer-can-make-jekyll-blog/)
