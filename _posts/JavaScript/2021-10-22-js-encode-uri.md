---
layout: post
title: "JavaScript - encodeURIComponent() & encodeURI()"
date: 2021-10-22
categories: JavaScript
---

`encodeURIComponent()` 과 `encodeURI()` 는 모두 URI 에 포함될 문자들을 UTF-8로 인코딩해주는 JavaScript 내장 함수들이다.

### encoding 이 필요한 이유?

Browser 의 URI 에 입력될 문자를 encoding 하는 이유는 바로 `/`, `&`, `space`, `!` 와 같은 문자들이 URI 상에서 특정한 역할을 하는 **예약어**이기 때문이다.

예를 들어, user 로부터 검색어를 입력 받아 server 로 `GET` 요청을 보내 결과를 응답해주는 Frontend App 이 있다고 가정해보자. 이 때 user 가 '`&foo=test`' 와 같은 예약어 문자들을 검색어로 입력하고 encoding 없이 URI 에 포함되게 된다면, 해당 문자들은 검색어가 아니라 하나의 URI parameter 처럼 동작하게 될 것이다.

이러한 이슈를 방지하기 위해 검색어와 같이 사용자가 자유롭게 입력하고 `GET` 요청으로 처리하는 URI parameter 는 **Frontend 에서 encode 하고 Backend 에서 decode 하여 사용하는 것이 안전하다.**

### 예시 코드

**우선 HTML 을 작성해보자.**

아래와 같이 encoding 을 하는 input 과 하지 않는 input 을 따로 생성해준다.

```html
<body>
    <input type="text" name="input" id="input"/>
    <button id="btn">submit</button>

    <br /><br />

    <input type="text" id="input-enc" />
    <button id="btn-enc">encode and submit</button>

    <script type="text/javascript" src="test2.js"></script>
</body>
```

![Untitled 1](https://user-images.githubusercontent.com/52827441/163089903-fdea6867-f996-4bd4-ab7e-b7cc038f47c3.png)

**JavaScript 를 살펴보자.**

우선 각각의 button 과 input element 들에게 DOM 을 통해 접근해 변수에 할당해준다.

```jsx
const btn = document.getElementById('btn');
const input = document.getElementById('input');

const btnEncoding = document.getElementById('btn-enc');
const inputEncoding = document.getElementById('input-enc');
```

그리고 서버에 `GET` 요청을 보내는 비동기 함수 두개를 선언해준다. 각각 URI 에 `search` 라는 param 을 추가해주며, input 에 입력된 값을 value 로 넣어준다. 이 때 한 input 은 그대로 URI 에 추가되고, 다른 하나는 encoding 하여 추가된다.

```jsx
async function getSomething({ input }) {
    await fetch(`?search=${input}`).then((res) => console.log('success', res));
}

async function getSomethingWithEncodedInput({ input }) {
    const encodedInput = encodeURIComponent(input);
    await fetch(`?search=${encodedInput}`).then((res) => console.log('success', res));
}
```

그런 다음, 위에서 선언한 input element 들과 get function 을 매개변수로 받아 호출하는  `click` 이벤트 핸들러를 선언해준다.

```jsx
const handleClick = (inputElement, getFunc) => () => {
    console.log(inputElement.value);
    try {
        getFunc({ input: inputElement.value });
    } catch (error) {
        console.error(error);
    }
}
```

마지막으로 각각의 버튼에 알맞는 이벤트 핸들러를 추가해주면 완성이다.

```jsx
btn.addEventListener('click', handleClick(input, getSomething));
btnEncoding.addEventListener('click', handleClick(inputEncoding, getSomethingWithEncodedInput));
```

**이제 결과를 살펴보자.**

![Untitled 2](https://user-images.githubusercontent.com/52827441/163089907-fd7952da-84e7-4339-a7cb-2c92a427e8a4.png)

두 input 모두 `"searchValue&foo=far"` 라는 input 을 동일하게 입력 받았지만,

위의 input 은 encoding 이 되지 않아 그대로 URI 에 추가되었고 다른 input 은 encoding 이 된채로 추가된 것을 확인할 수 있다.

만약 encoding 이 안된 URI 가 서버로 전달된다면 `search` 라는 param 의 값은 `"searchValue"` 로만 인식이 되고 뒤의 `"&foo=far"` 는 새로운 param 의 key, value 로 인식될 것 이다.

### 또 다른 사용법

![Untitled 3](https://user-images.githubusercontent.com/52827441/163089909-2d12b9fc-fd65-457f-9104-a9ed801207ec.png)

위처럼 JS 객체 형태의 값을 간단하게 하나의 URI param 의 value 로 추가하기 위해서도 encoding 을 이용한다.

(`encodeURIComponent()` 와 `encodeURI()` 는 string 을 매개변수로 받으므로 해당 JS 객체를 `JSON.stringfy()` 로 string 화 해주어 encoding 하고 있다.)

### `encodeURIComponent()` 와 `encodeURI()` 의 차이점?

`encodeURIComponent()` 는 URI 에 포함되는 일부를 encoding 할 때 사용하는 함수이고

`encodeURI()` 는 URI 나 URL 전체를 encoding 할 때 사용된다.

### References

- [https://www.freecodecamp.org/news/javascript-url-encode-example-how-to-use-encodeuricomponent-and-encodeuri/](https://www.freecodecamp.org/news/javascript-url-encode-example-how-to-use-encodeuricomponent-and-encodeuri/)
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/encodeURI](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/encodeURI)
- [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)
