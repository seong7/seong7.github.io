---
layout: post
title: "React 프로젝트 - Dark Theme 적용기"
date: 2020-07-23
author: Jason
categories: react
published: false
---

## 개요

`Dark Theme` 은 기존의 밝은 색 조합으로 이루어진 서비스를 어두운 색의 모드로도 사용할 수 있게 해주는 UX 요소이다.

원래 모바일 앱에서 처음 시작되었지만, 요즘에 Facebook, Naver 등 대형 웹 사이트에서 적용이 되며 웹에서도 유행처럼 번지고 있다.

그리고 이번에 나도 드디어 개인 프로젝트에 `Dark Theme` 을 적용할 수 있었다.

`Dark Theme` 을 적용하는 다양한 방법이 있지만 내가 선택한 방법은 CSS 변수를 사용하는 방법이다.

어떻게 기능을 구현하게 되었는지 자세한 과정을 풀어보려 한다.

<br>

## 구현 방법 조사

최근 `Dark Theme` 이 구현된 사이트를 만날 때 마다 반가운 마음으로 HTML, CSS source 를 살펴 봤다.

그리고 기본적으로 HTML 과 CSS 를 사용해 기능을 구현하며 **각각의 역할** 이 있다는 것을 알게되었다. 그리고 그 **역할을 수행하기 위한 방법을 두가지씩** 찾을 수 있었다.

- **HTML** : 현재 페이지가 `dark` 모드인지 `light` 모드인지 구분한다.

  **방법 1** : dataset 속성으로 구분  
  **방법 2** : class 로 구분

- **CSS** : `dark` 모드에 적용할 style 과 `light` 모드의 style 각각 선언한다.

  **방법 1** : 선택자로 구분해 각각의 style 작성  
  **방법 2** : CSS 변수와 var() 함수 사용

사실 HTML 의 경우 어떤 방법을 선택하든 코드에 큰 영향을 끼치진 않는다.  
dataset 속성을 다루느냐 class 를 다루느냐의 차이일 뿐.

**중요한 것은 CSS 이다.**  
어떤 방법을 따르느냐에 따라 작성하는 코드의 모양이 크게 차이나기 때문이다.

기존 웹사이트들을 예시로 각 방법들의 장단점을 파악해봤다.

### 1. CSS 선택자를 활용한 방법 (HTML : dataset 속성 사용)

**대표 적용 사이트 : [Naver](www.naver.com)**

  <a href="https://user-images.githubusercontent.com/52827441/88282552-c1133680-cd24-11ea-8878-4dda02981ae8.PNG" data-lightbox="naver-light-large" data-title="Naver - Light Theme">
  <img src="https://user-images.githubusercontent.com/52827441/88282552-c1133680-cd24-11ea-8878-4dda02981ae8.PNG" alt="Naver - Light Theme" style="height: 350px; width: auto;"/>
  </a>

  <a href="https://user-images.githubusercontent.com/52827441/88282551-bf497300-cd24-11ea-966d-d8f1f912aef4.PNG" data-lightbox="naver-dark-large" data-title="Naver - Dark Theme">
  <img src="https://user-images.githubusercontent.com/52827441/88282551-bf497300-cd24-11ea-966d-d8f1f912aef4.PNG" alt="Naver - Dark Theme" style="height: 350px; width: auto;"/>
  </a>

네이버의 각 Theme 모드 별 화면이다.

**특징은 다음과 같다.**

1. `<html>` 태그에 `data-dark` 라는 attribute<sub>(속성)</sub> 를 선언해주고 각각 `true` 와 `false` 로 구분하고 있다.

   > 참고로 JavaScript 에서 `data-xx` 로 이름 붙인 속성에 접근하는 방법은 `{해당 html 요소}.dataset.xx` 이다.

2. CSS 는 Light Theme 이 default 로 작성되었고 `html[data-dark=true]` <sub>(\<html> 태그 중 `data-dark=true` 속성이 있는 요소를 지정하는 CSS 선택자)</sub>의 하위 컴포넌트들에 대해 스타일을 재선언 해주고 있는 듯 하다.

3. Theme 모드를 변경할 때 `reload` 가 발생한다.  
   <sub>이건 왜 이렇게 구현했는지 이해가 잘되지 않는다. <strong>화면이 다시 reset 되는게 UX 면에서 그다지 좋아보이지 않았다.</strong> 궁금하긴 하지만 너무 깊게 파고 들만큼 중요한 부분은 아니라고 생각한다. 단순히 'CSS 만 변경되는 것이 아닐 수 있겠다' 고 추측하고 넘어가기로 했다.</sub>

<br>

**다음은 장점과 단점이다.**

- **장점**

  - IE 에서도 `Dark Theme` 을 사용할 수 있다.  
    <sub>IE 사용률이 높은 한국의 대표 포털사이트 Naver 가 해당 방법을 선택한 이유이지 않을까 싶다.</sub>

- **단점**

  - CSS 에서 `html[data-dark=true]` 의 하위 요소들을 하나하나 재지정해가며 style 을 작성해야한다.

    예를 들어, (scss 문법)

    ```css
    html[data-dark="false"] {
      #header {
        /* style... */
      }
      #main {
        /* style... */
      }
    }

    html[data-dark="true"] {
      #header {
        /* style... */
      }
      #main {
        /* style... */
      }
    }
    ```

    **HTML 요소들을 하나하나 새로 지정하며 `dark theme` 에서 변경될 style 을 작성해줘야한다.**  
    컴포넌트 트리가 깊어지면 어떤 컴포넌트의 style 을 변경해줄지도 하나하나 선택해야하므로 꽤나 복잡한 작업이 될 것이다.

    내가 적용할 사이트 또한 컴포넌트가 적지 않기 때문에 분명 엄청난 양의 반복 작업이 될 것 같았다.  
    <sub>물론 Naver 에선 내가 떠올리지 못한 smart 한 방법으로 작성했겠지만, 난 떠올릴 수 없었다.. </sub>

<br>

### 2. CSS 변수와 var() 함수 사용 (HTML: class 사용)

**대표 적용 사이트 : [Overreacted 블로그](https://overreacted.io/)**

Overreacted 는 **Facebook React 팀의 개발자 [Dan Abramov 님](https://github.com/gaearon)** 의 개인 기술 블로그이다.  
우연히 알게된 이 블로그에서도 Dark Theme 을 찾을 수 있었다.

<a href="https://user-images.githubusercontent.com/52827441/88282542-ba84bf00-cd24-11ea-9f74-4c2773e7629e.PNG" data-lightbox="dan-light-large" data-title="Dan's blog - Light Theme">
<img src="https://user-images.githubusercontent.com/52827441/88282542-ba84bf00-cd24-11ea-9f74-4c2773e7629e.PNG" alt="Dan's blog - Light Theme" style="height: 350px; width: auto;"/>
</a>

<a href="https://user-images.githubusercontent.com/52827441/88282536-b9539200-cd24-11ea-9ab8-c94425118fc2.PNG" data-lightbox="dan-dark-large" data-title="Dan's blog - Dark Theme">
<img src="https://user-images.githubusercontent.com/52827441/88282536-b9539200-cd24-11ea-9ab8-c94425118fc2.PNG" alt="Dan's blog - Dark Theme" style="height: 350px; width: auto;"/>
</a>

**이 블로그의 Dark Theme 구현 방법 특징은 아래와 같다.**

1. `<body>` 태그의 `class` 로 `dark` 와 `light` 를 구분하고 있다.

2. prefix `--` 가 붙은 CSS 변수를 사용하고 있다.

3. `transition` 을 적용해 색 전환이 부드러워 UX 면에서 뛰어나다.

[Facebook](www.facebook.com) 웹사이트 또한 이런 방식으로 구현되어 있다.

<br>

**다음은 장점과 단점이다.**

- **장점**

  - CSS 변수를 사용해 각 `Theme` 에 따라 변수의 값만 변경해주었다.  
    즉, 하위 html 요소들을 재지정하며 style 을 다시 작성할 필요가 없다.

- **단점**

  - 해당 CSS 문법은 IE 에서 지원하지 않는다.

<br>

## Cross Browsing (IE)

이미 언급한대로 위의 두 방법은 CSS 문법의 IE 지원여부가 갈린다.  
그럼 예시로 든 사이트들은 IE 에서 어떻게 서비스하고 있을까?

**1. Naver**

<a href="hhttps://user-images.githubusercontent.com/52827441/88371728-8fa37500-cdcf-11ea-9758-81f55b9a5f09.png" data-lightbox="dan-dark-large" data-title="Dan's blog - Dark Theme">
<img src="https://user-images.githubusercontent.com/52827441/88371728-8fa37500-cdcf-11ea-9758-81f55b9a5f09.png" alt="Dan's blog - Dark Theme" style="height: 350px; width: auto;"/>
</a>

**2. Facebook**

**3. Overreacted**

<br>

## React 프로젝트에 적용하기

나는 **CSS 변수를 사용한 방법** 으로 프로젝트에 `Dark Themem` 을 구현하기로 했다.  
CSS 변수를 사용해보고 싶기도 했고 유지보수성 또한 뛰어나기 때문이다.

<br>

### 1. CSS 변수 및 var() 함수

지금까지 CSS 변수 문법이 존재하는지 몰랐기 때문에 조금의 러닝 커브가 필요했다.  
그리 어렵지는 않다.

구체적인 개념은 [W3S](https://www.w3schools.com/css/css3_variables.asp) 에 잘 정리되어 있지만 간략히만 정리하자면,

- IE 는 해당 문법을 지원하지 않는다.

- CSS 변수는 반드시 two dashes (`--`) 로 시작해야한다.

- CSS 의 `var()` 함수로 변수를 호출할 수 있다.

  ```css
  /* var() 표기법 */
  var(--변수, value)

  /* --변수 : required. --변수에 저장된 값이 적용된다. */
  /* value : optional. --변수 가 유효하지 않을 경우 적용되는 fallback value 이다. */
  ```

#### 사용 예제

- **variables.css**  
  각 Theme mode 에 따라 CSS 변수를 저장한 CSS 파일이다.

```css
/* body 태그 중 data-theme 속성 값으로 light 를 가진 것 선택 */
body[data-theme="light"] {
  /* common */
  --bg: #fff;

  /* input */
  --bg-input: #fff;
  --border-input: #2bb14c;

  /* text */
  --color-text: #333333;
  --color-green: #2bb14c;
}

/* body 태그 중 data-theme 속성 값으로 dark 를 가진 것 선택 */
body[data-theme="dark"] {
  /* common */
  /* --bg: #121212; */
  --bg: #303a52;

  /* input */
  --bg-input: #383838;
  --border-input: #2bb14c;

  /* text */
  --color-text: #e5e5e5;
  --color-green: #2bb24c;
}
```

- **index.css**  
  위에 선언한 변수들을 호출해 적용하는 부분이다.

```css
body {
  background: var(--bg);
  color: var(--color-text);
}

input {
  background: var(--bg-input)
  border: 1px solid var(--border-input)
}
```

이런식으로 변수를 호출하며 컴포넌트를 styling 하면 된다.  
색을 바꿔줄 때도 `variables.css` 만 수정해주면 되므로 유지보수가 간편하다.

<br>

### 2. Toggle UI 구현

- **Toggle.jsx**  
  재사용 가능한 Toggle 버튼 컴포넌트

```javascript
import React from "react";
import "./Toggle.scss";

const Toggle = (props) => {
  return (
    <>
      <input
        type="checkbox"
        id="switch"
        className="toggle"
        onChange={props.onChange}
        checked={props.isChecked}
      />
      <label htmlFor="switch" className="toggle-label">
        {props.text /* toggle 의 이모티콘 넣어주는 곳 */}
      </label>
    </>
  );
};

export default Toggle;
```

- **Toggle.scss**

```css
.toggle {
  height: 0;
  width: 0;
  visibility: hidden;
}

.toggle-label {
  cursor: pointer;
  width: 44px;
  height: 27px;
  background: #fefefe;
  display: block;
  border-radius: 100px;
  position: relative;
  bottom: 15px;

  svg {
    color: black;
    position: relative;
    top: 5px;
    left: 2px;
    font-size: 18px;
  }
}

.toggle-label:after {
  content: "";
  position: absolute;
  top: 1.6px;
  left: 2px;
  width: 24px;
  height: 24px;
  background: #080827;
  border-radius: 90px;
  transition: 0.3s;
}

.toggle:checked + .toggle-label:after {
  left: calc(100% - 2px);
  transform: translateX(-100%);
}
```

<br>

### 3. Theme 전환 로직 구현

- **ThemeToggle.jsx**  
  위에서 생성한 Toggle.jsx 를 사용해 theme 을 제어하는 로직을 가진 컴포넌트를 생성한다.

```javascript
import React, { useCallback, useState, useEffect } from "react";
import { Toggle } from "../../common";

// <body> 에 색변환 transition 을 주는 class 를 추가했다가 제거하는 함수
const themeTransition = () => {
  document.body.classList.add("theme-transition");
  setTimeout(() => {
    document.body.classList.remove("theme-transition");
  }, 1000);
  /*
     css 에 아래를 추가 한다. (theme-transition 클래스에 대한 style)
   
     body.theme-transition,
     body.theme-transition *,
     body.theme-transition *:before,
     body.theme-transition *:after {
       transition: all 0.3s ease-in-out !important;
       transition-delay: 0 !important;
     }

   */
};

// <body> 에 data-theme 속성을 set 해주는 함수
// localStorage 에 저장된 value 를 받아와 매개변수로 호출한다. (default 는 'light' 이다.)
const setBodyDataTheme = (value) => {
  document.body.setAttribute("data-theme", value || "light");
};

// toggle UI 에 넣을 icon UI 를 생성해주는 함수
// (ReactIcon 모듈을 받아서 사용했다.)
const themeIcons = (
  <>
    <ReactIcon icon={"IoIosMoon"} /> <ReactIcon icon={"WiDaySunny"} />
  </>
);

// React Componenet
const ThemeToggle = () => {
  const [theme, setTheme] = useState(""); // 'dark' or 'light'

  // Toggle UI 의 onChange event handler 함수
  const handleChange = useCallback((e) => {
    themeTransition(); // transition class 제어

    // toggle (input - checkbox) 이 체크되었는지에 따라
    // state 인 theme 의 값을 set 한다.
    if (e.target.checked) {
      setTheme("dark");
    } else if (!e.target.checked) {
      setTheme("light");
    }
  }, []);

  // 컴포넌트 최초 렌더 시 실행
  useEffect(() => {
    // 최초 렌더할 때 localStorage 에서 값을 받아온다.
    const _theme = localStorage.getItem("theme");
    setTheme(_theme); // set state
  }, []);

  // 컴포넌트 최초 렌더 시 실행되고 (위의 useEffect 다음에 실행된다.)
  // 그 외 렌더 시에는 state 인 theme 값에 변화가 있으면 실행된다.
  useEffect(() => {
    // theme 값이 변화되었으므로 localStorage 에 새로 저장한다.
    localStorage.setItem("theme", theme);
    setBodyDataTheme(theme); // <body> 의 data-theme 속성 값을 변경한다.
  }, [theme]);

  return (
    <Toggle
      onChange={handleChange}
      text={themeIcons}
      isChecked={theme === "dark"}
    />
  );
};

export default ThemeToggle;
```

<br>

### 4. localStorage 사용

<br>

## 출처

- [네이버 웹 홈페이지](www.naver.com)

- [Dan Abramov 님의 Overreacted 블로그](https://overreacted.io/)

- [W3S - CSS variable](https://www.w3schools.com/css/css3_variables.asp)
