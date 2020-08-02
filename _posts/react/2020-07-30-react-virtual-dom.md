---
layout: post
title: "[React 학습] - React 의 동작 원리 (가상 DOM)"
date: 2020-07-30
author: Jason
categories: react
published: true
---

## 개요

React 를 사용하는데만 집중해왔는데 좀 더 깊은 학습을 위해 개발자 [도디](https://dodody.github.io/)님과 함께 매주 React 에 관련된 주제를 하나씩 정해 학습하고 글을 publish 하는 스터디를 시작했다.

첫 번째이자 이번 주 주제는 **React 의 동작 원리** 이다.

<br>

## Virtual DOM (가상 DOM)

React 의 장점은 가상 DOM 을 통해서 UI 를 빠르게 업데이트한다는 점이다.

가상 DOM 은 **이전 UI 상태를 메모리에 유지**해서, **변경될 UI 의 최소 집합을 계산**하는 기술이다.

즉, 가상 DOM 덕분에 불필요한 UI 업데이트는 줄고 성능은 좋아진다.

React 는 가상 DOM 을 두어 아래 **두 가지의 포인트**를 향상시켰다.

1. **개발의 편의성 (DOM 을 직접 제어하지 않는다.)**  
   개발자는 직접 DOM 을 제어하지 않고 가상 DOM 을 제어한다. 대신 React 가 가상 DOM 을
   실제 DOM 에 반영하는 작업을 한다.

   > DOM 을 직접 업데이트하는 코드는 잘 관리하지 않으면 프로그램이 커질수록 복잡도가 기하급수적으로 증가한다. 따라서 Vanilla JavaScript 로 UI 업데이트를 처리하려면 React 에 상응하는 자체 라이브러리를 만들어서 관리하는게 좋다.

2. **성능**  
   DOM 직접 조작은 효율적이지 않다. <sub>([설명 링크](https://velopert.com/3236))</sub>

또 다른 JavaScript 프레임워크인 **[Vue.js](https://vuejs.org/)** 에서도 React.js 의 **가상 DOM 개념**을 사용하고 있다. 뿐만 아니라 앞으로 나올 프레임워크들에도 가상 DOM 이 도입될 것으로 예상되고 있다.

**즉, 가상 DOM 을 이해하면 Vue 를 포함한 미래의 프레임워크들의 동작도 쉽게
이해할 수 있을 것이다.**

이 글에서 알아볼 내용은 다음과 같다.

**1. 가상 DOM 을 이루고 있는 React 요소란?**  
**2. 가상 DOM 은 어떻게 만들어지는가?**  
**3. 가상 DOM 은 어떻게 변화를 감지하는가?**

<br>

## 가상 DOM 을 이루고 있는 React 요소란?

**리액트 요소 (React Element)** 는 리액트가 UI 를 표현하는 수단이다.  
리액트는 **리액트 요소로부터 가상 DOM 을 만들고**, 실제 DOM 에 반영할 변경 사항을 찾는다.

리액트 요소는 `React.createElement()` 함수로부터 반환된다.

```javascript
// jsx 코드
const reactElement = <a key='1' href='https://www.github.com'>
                      click
                     </a>;

// jsx 코드가 변환된 상태
const reactElement = React.createElement(
  'a',
  { href: 'https://www.github.com' },
  'click'
);

// console 에 해당 리액트 요소를 출력한 결과
const consoleLogged = {
  type: 'a', // 문자열
  key: 'key1',
  ref: null
  props: {  // key 와 ref 를 제외한 모든 속성은 props 에 들어간다.
    href: 'https://www.github.com',
    children: 'click'
  }
}
```

여기서 **요소가 HTML 태그로 감싸져 있으면 type 속성의 값이 문자열이고 리액트 컴포넌트로 감싸져 있으면 함수가 된다.**

여기서 왜 리액트 컴포넌트가 하나의 요소로 둘러 쌓여야 하는지 알 수 있다.
바로 **하나의 type 속성값**을 가져야하기 때문이다.

그리고 리액트 요소는 불변 객체이기 때문에 속성값을 변경할 수 없다.  
(`reactElement.type = 'b'` : error)

다음 주제에서 가상 DOM 이 정확이 무엇인지 알아보고  
왜 리액트 컴포넌트들의 구조를 '컴포넌트 Tree' 라고 부르는지도 알아보자.

<br>

## 가상 DOM 은 어떻게 만들어지는가?

하나의 화면을 표현하기 위해 여러 개의 리액트 요소가 Tree 구조로 구성된다.  
**여기서 리액트 요소의 Tree 가 바로 가상 DOM 이다.**

이게 무슨 말인지 좀 더 자세히 알아 보자.

**가상 DOM 은 리액트 요소로부터 만들어진다.**  
리액트는 렌더링을 할 때마다 가상 DOM 을 만들고 이전의 가상 DOM 과 비교한다. 이는 실제 DOM 의 변경사항을 최소화하기 위한 과정이다.

아래 코드는 할 일의 우선순위를 state 로 관리하는 리액트 컴포넌트이다.  
이 코드를 기반으로 **리액트 요소가 실제 DOM 으로 만들어지는 과정**을 이해해보자.

```javascript
function Todo({ title, desc }) {
  const [priority, setPriority] = useState("hight");

  function onClick() {
    setPriority(priority === "high" ? "low" : "high");
  }

  return (
    <div>
      <Title title={title} />
      <p>{desc}</p>
      <p>{priority === "high" ? "우선순위 높음" : "우선순위 낮음"}</p>
      <button onClick={onClick}>우선순위 변경</button>
    </div>
  );
}

const Title = React.memo(({ title }) => {
  return <p>{title}</p>;
});

ReactDOM.render(
  <Todo title="리액트 공부" desc="리액트 교재 읽기" />,
  document.getElementById("root")
);
```

컴포넌트 설명

- `Todo` 컴포넌트는 `Title` 컴포넌트를 자식으로 사용한다.
- 버튼을 클릭하면 state 인 priority 값이 변경되고 화면이 re-render 된다.
- `React.memo` 로 만들어진 `Title` 컴포넌트는 속성값이 변경될 때만 호출된다.

`ReactDOM.render` 함수 <sub>CRA 에선 `index.js` 에서 호출되어 있다.</sub> 로 전달된 위 리액트 요소의 Tree 는 다음과 같다.

```javascript
const initialEementTree = {
  type: Todo, // 함수
  props: {
    title: "리액트 공부하기",
    desc: "리액트 교재 읽기",
  },
  // ...
};
```

여기서 Todo 컴포넌트의 렌더링 결과를 얻기 위해 Todo 컴포넌트 함수를 호출한다.

```javascript
const elementTree = {
  type: "div", // Todo => 'div'
  props: {
    children: [
      {
        type: Title, // 함수
        props: { title: "리액트 공부하기" },
        // ...
      },
      {
        type: "p",
        props: { children: "리액트 교재 읽기" },
        // ...
      },
      {
        type: "p",
        props: { children: "우선순위 높음" },
        // ...
      },
      {
        type: "button",
        props: {
          onClick: function () {
            /* Todo 컴포넌트의 onClick 함수 */
          },
          children: "우선순위 변경",
        },
      },
    ],
  },
};
```

이렇게 보니 왜 리액트 요소의 구조를 Tree 라고 하는지 이해가 된다.

**리액트 요소 Tree 가 실제 DOM 으로 만들어지기 위해서는 모든 리액트 요소의
type 속성값이 문자열이어야한다.**  
type 속성값이 문자열이어야 해당 요소를 HTML 태그로 변환할 수 있기 때문이다.

**즉, 모든 컴포넌트 함수가 호출되어야 한다.**

Tree 에 남아 있는 컴포넌트 함수 `Title` 을 호출한다.

```javascript
const elementTree = {
  type: "div",
  props: {
    children: [
      {
        type: "p", // Title => 'p'
        props: { children: "리액트 공부하기" },
        // ...
      },
      {
        type: "p",
        props: { children: "리액트 교재 읽기" },
        // ...
      },
      {
        type: "p",
        props: { children: "우선순위 높음" },
        // ...
      },
      {
        type: "button",
        props: {
          onClick: function () {
            /* Todo 컴포넌트의 onClick 함수 */
          },
          children: "우선순위 변경",
        },
      },
    ],
  },
};
```

이제 모든 리액트 요소들의 type 속성값이 문자열이므로 실제 DOM 을 만들 수 있다.

**이와 같이 실제 DOM 을 만들 수 있는 리액트 요소 Tree 를 가상 DOM 이라고 한다.**

즉, 가상 DOM 은 최초의 리액트 요소 Tree 에서 모든 리액트 함수 컴포넌트를 호출한 상태이다.

여기까지는 **최초에 `ReactDOM.render()` 함수에 의해 시작된 렌더 단계**를 살펴봤다.

<br>

## 가상 DOM 은 어떻게 변화를 감지하는가?

리액트에서 화면 업데이트의 단계는 렌더 단계 (render phase 또는 [reconciliation](https://en.dict.naver.com/#/search?range=all&query=reconciliation) phase ) 와 커밋 단계 (commit phase) 를 거친다.

- **렌더**는 실제 DOM 에 반영할 변경 사항을 파악하는 단계이다.
- **커밋**은 파악된 변경 사항을 실제 DOM 에 반영하는 단계이다.

여기서는 **상태값 변경 함수 (setState) 에 의해 렌더가 발생하는 단계**를 알아보자.

브라우저에서 실제 DOM 을 변경하는 작업은 다른 작업에 비해 오래 걸리기 때문에 꼭
필요한 부분만 변경하는 것이 중요하다.

즉, **변경된 부분을 잘 찾아내는 것이 중요하다.**

위의 코드에서 `onClick` 함수를 통해 `setPriority()` 가 호출된 후의 리액트 요소 Tree 를 살펴보자.

```javascript
const elementTree = {
  type: "div",
  props: {
    children: [
      {
        type: Title, // 함수
        props: { title: "리액트 공부하기" },
        // ...
      },
      {
        type: "p",
        props: { children: "리액트 교재 읽기" },
        // ...
      },
      {
        type: "p",
        props: { children: "우선순위 낮음" }, // 높음 => 낮음
        // ...
      },
      // 아래 코드는 같음

```

`Title` 컴포넌트는 `React.memo` 로 만들어졌기 때문에 속성값이 변하지 않아 호출되지 않고 이전 **가상 DOM 에서의 호출 결과**가 재사용된다.

즉, 위의 코드가 새로운 가상 DOM 이다.  
따라서 실제 DOM 에 '우선순위 낮음' 문자열을 포함하는 p 태그만 수정된다.

## 남은 의문점

지금까지 리액트 요소를 이용해 렌더 단계를 알아봤다. 하지만 엄밀히 말하면 리액트 요소는
**[파이버 (fiber) 라는 구조체](https://stackoverflow.com/questions/45341423/what-is-difference-between-react-vs-react-fiber)**로 변환된다. 파이버는 리액트 버전 16부터 도입된 구조체
이름이다.

파이버 또한 type 과 props 값을 가지며 모든 type 속성값이 문자열이 될 때까지 연산된다는 사실에는 변함이 없다.

**파이버에 대해서 더 알아볼 필요가 있을 것 같다.**

<br>

## Reference

- Naver D2 - [React 적용 가이드 - React 작동 방법](https://d2.naver.com/helloworld/9297403)

- [Virtual DOM 조작 방식의 장점](https://velopert.com/3236)

- [(책) 실전 리액트 프로그래밍 - 이재승](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=243686179)
