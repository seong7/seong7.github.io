---
layout: post
title: "React 프로젝트 - Dark Theme 적용기"
date: 2020-07-23
author: Jason
categories: react
---

## Dark Theme 적용하기

### 1. Toggle 컴포넌트 생성

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

- **ThemeToggle.jsx**  
  Toggle.jsx 를 사용해 theme 을 제어하는 로직을 가진 컴포넌트를 생성한다.

```javascript
import React, { useCallback, useState, useEffect } from "react";
import { Toggle, ReactIcon } from "../../common";

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

## 2. CSS 변수 선언 후 var() 함수로 호출

지금까지 CSS 변수가 존재하는지 몰랐기 때문에 조금의 러닝 커브가 필요했다.  
하지만 생각보다 어렵지 않았다.

구체적인 개념은 [W3S](https://www.w3schools.com/css/css3_variables.asp) 에 잘 정리되어 있지만 간략히만 정리하자면,

- IE 는 해당 문법을 지원하지 않는다.

- CSS 변수는 반드시 two dashes (`--`) 로 시작해야한다.

- CSS 의 `var()` 함수로 변수를 호출할 수 있다.

  ```
  /* 표기법 */
  var(--변수, value)

  /* --변수 : required. --변수에 저장된 값이 적용된다. */
  /* value : optional. --변수 가 유효하지 않을 경우 적용되는 fallback value 이다. */
  ```

### 사용 예제

- **variables.css**  
  각 Theme mode 에 따라 CSS 변수를 저장한 코드이다.

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
  위에 선언한 변수들을 호출해 적용

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
