---
layout: post
title: "[React 학습] - React 의 동작 원리"
date: 2020-07-30
author: Jason
categories: react
published: true
---

## 개요

React 를 사용하는데만 집중해왔는데 좀 더 심도 깊은 학습을 위해 개발자 [도디](https://dodody.github.io/)님과 함께 매주 React 에 관련된 주제를 하나씩 정해 글을 post 하는 스터디를 시작했다.

첫 번째이자 이번 주 주제는 **React 의 작동 원리** 이다.

## Virtual DOM

React 는 직접 DOM 을 제어하는 방식이 아니라 중간에 가상의 DOM 인 Virtual DOM 을 두어 아래 두 가지의 포인트를 향상시켰다.

1. 개발의 편의성 (DOM 을 직접 제어하지 않는다.)
2. 성능 (배치 처리<sub>setState 와 관련있는 듯 하다.</sub>로 DOM 변경 <sub>DOM 직접 조작은 효율적이지 않다. ([설명 링크](https://velopert.com/3236))</sub>)

Virtual DOM 이 바로 React 의 가장 주요 장점인데 어떻게 만들어지고 관리되는지 알아보겠다.

## Virtual DOM 렌더링

- Virtural DOM 은 실제 DOM 의 구조와 비슷한, React 객체의 트리이다.
- 개발자는 직접 DOM 을 제어하지 않고 Virtual DOM 을 제어한다.
- React 는 Virtual DOM 을 실제 DOM 에 반영하는 작업을 한다.

다음과 같이 [JSX 문법](https://reactjs.org/docs/jsx-in-depth.html)을 사용한 [ReactDOM.render() 함수](https://github.com/naver/react-sample-code/blob/master/src/index.js#L18-L23)를 호출하면 Virtual DOM 을 만들기 시작한다.

```javascript
ReactDOM.render(<App />, document.getElementById("root"));
```

위 코드의 JSX 문법을 변환하면 다음과 같은 JavaScript 코드가 만들어진다.

```javascript
ReactDOM.render(React.createElement(App), document.getElementById("root"));
```

`render()` 함수를 호출할 때 `React.createElement(App)` 객체를 parameter 로 전달하며,
`render()` 함수는 [React 에서 사용하는 타입의 컴포넌트](https://github.com/facebook/react/blob/6eebed0535a5e28effb7200783cede7a4bdab7ec/src/renderers/shared/stack/reconciler/instantiateReactComponent.js#L71-L138)<sub>링크의 코드를 봐도 잘 모르겠다..</sub>를 생성한다.

이때 생성하는 컴포넌트는 주로 `ReactCompositeComponent` 객체와 `ReactDOMComponent` 객체다.

- `ReactCompositeComponent` 객체는 DOM 이 아닌 컴포넌트를 생성할 때 사용된다.
- `ReactDOMComponent` 객체는 DOM 을 만들 때 생성하는 컴포넌트다.

**`ReactCompositeComponent` 객체의 주요 작업**

- `constructor()` 메서드 실행
- `componentWillMount()` 메서드 실행
- 렌더링 실행
- 배치 처리 작업 (`ReactReconcileTransaction` 객체)에 메서드나 속성 등록
  - `componentDidMount()` 메서드가 있으면 `ComponentDidMount()` 메서드 등록
  - ref 속성이 있으면 attachRefs 속성 등록
- 하위 `ReactComponent` 객체가 있으면 `ReactCompoenent` 객체를 생성하고 다시 `ReactReconciler.mountComponent()` 메서드를 실행

## Reference

- Naver D2 - [React 적용 가이드 - React 작동 방법](https://d2.naver.com/helloworld/9297403)

- [Virtual DOM 조작 방식의 장점](https://velopert.com/3236)
