---
layout: post
title: "React-Redux 라이브러리"
date: 2020-09-24
categories: react
published: true
---

# React-Redux 라이브러리

📖 Table of Contents

- **개요**
- **제공되는 함수**

  - connect()

- **Hooks**

  - useSelector()
  - useDispatch()
  - useStore()
  - Custom Context

- **Reference**



## 개요

React Redux 는 **React 의 Redux 를 위한 공식적인 `UI binding library`** <sub>(어떠한 로직과 UI 를 결합시키는 라이브러리)</sub> 이다.

즉, Redux 와 React 를 함께 쓰고 있다면, **React-Redux** 라이브러리를 사용해 두 라이브러리를 bind 해줘야한다.

`UI` 와 Redux 를 연결하는 logic 을 직접 구현할 수도 있겠지만, 반복작업이 상당히 많을 것이고 그와 동시에 **UI performance** 를 최적화하는 것은 매우 복잡할 것이다.

**간단히 말해** React-Redux 는 **React 컴포넌트 (UI) 가** `Redux store` 로부터 data 를 읽을 수 있도록 해주고 `action` 을 `dispatch` 할 수 있게 해준다.

<br />

## 제공되는 함수

### `connect()`

Hook 이 나오기 전 `store` 와 component 를 연결하기 위해 사용하던 함수이다.

`connect()` 로 `store` 와 연결되는 컴포넌트를 **HOC** (Higher Order Component) 라고 부른다.

```javascript
import { connect } from "react-redux";
import { increment, decrement, reset } from "./actionCreators";

const Counter = ... // React Component

const mapStateToProps = (state /*, ownProps*/) => {
  return {
    counter: state.counter,
  };
};

const mapDispatchToProps = { increment, decrement, reset };

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```



## Hooks

**React Redux v7.1.0.** 에서 함수형 컴포넌트에서 더 간편한 방법으로 `store` 에 접근할 수 있도록 다양한 **hook** 들이 release 되었다.

hook 들이 나오면서 더 이상 복잡한 HOC 사용 없이도 `Redux store` 구독과 `dispatch actions` 를 할 수 있게 되었다.

단순히 원하는 hook 을 `import` 해서 함수형 컴포넌트 내부에서 사용하면 된다.



### - `useSelector()`

`Redux store` 로부터 원하는 data 를 추출해 `return` 시켜주는 hook 이다.

```typescript
const result: any = useSelector(selector: Function, equalityFn?: Function)
```

#### arguments

- **`selector`** - function

  `store` 의 data 를 추출하는 함수이다.

  임의의 순간에 잠재적으로 여러번 실행될 수 있기 때문에 반드시 **순수함수** 이어야 한다.

  `connect` 함수의 `mapStateToProps` 인자와 비슷한 역할을 한다.  
  `mapStateToProps` 와 차이점은 아래와 같다.

  - `object` 타입이 아닌 어떤 타입으로도 값을 추출할 수 있다.

  - `ownProps` 를 인자로 받지 않는다. 하지만 `closure` 또는 `curried selector` 형태로 컴포넌트의 `props` 에 접근할 수 있다.

    > 하지만, `props` 에 의존하는 것은 잠재적 문제가 될 수 있으므로 피하도록 하자 ([문서 참조](https://react-redux.js.org/api/hooks#usage-warnings))

  - 함수형 컴포넌트가 `render` 될때마다 실행된다.  
    단, `useSelector()` 는 performance 향상을 위해 **caching** 을 지원한다.  
     `selector` 함수의 reference를 이전 `render` 때와 비교해  
     **reference 가 다를 경우**에만 `selector` 함수를 실행시킨다.  
     **reference 가 동일할 경우엔** `selector` 실행 없이 **cache 된 data** 를 `return` 한다.

  - `action` 이 `dispatch` 된 경우, `useSelector()` 는 `selector` 를 실행시키고 이전 결과와 현재 결과 의 reference 를 비교를 한다. 만약 reference 가 다를 경우에만 **컴포넌트는 re-render** 된다.  
    (비교는 `strict` `===` 를 사용한다. / <sub>`connect()` 는 field 단위의 **[shallow equality check](https://stackoverflow.com/questions/36084515/how-does-shallow-compare-work-in-react)** 를 한다.</sub>)  
    **: `reducer` 에서 새로운 reference `state` 를 생성해줘야하는 이유 중 하나인 것 같다. (re-render 하기 위해)**

  - `connect()` 와 달리 `useSelector()` 는 부모 컴포넌트의 **re-rendering** 에 의한 **강제 re-rendering** 은 막지 못한다. (props 가 변하지 않았다 하더라도..)  
    : **Performance 최적화를 위해서는 `React.memo` 를 사용하는 것이 좋다.** [(링크)](https://react-redux.js.org/api/hooks#performance)

- **`equalityFn?`** - function

  `selector` 함수의 결과 값을 이전 값과 비교하는 **logic** 이다.  
  컴포넌트를 **re-render** 시킬 건지 판단하는 역할을 한다.

  `default` 는 **`strict` `===`** 가 사용된다.

  즉, `selector` 함수에서 **매번 새로운 객체를 생성해 `return`** 시키면 항상 re-render 가 일어나 **비효율적**이다.



#### 사용 예

- Basic usage:

  ```javascript
  import React from "react";
  import { useSelector } from "react-redux";

  export const CounterComponent = () => {
    const counter = useSelector((state) => state.counter);
    return <div>{counter}</div>;
  };
  ```

- `closure` 로 `props` 에 접근해 값을 추출

  > **!!! selector 를 pure 하게 써야 [error](https://react-redux.js.org/7.1/api/hooks#usage-warnings) 를 방지할 수 있으므로 이 방법은 사용하지 말자**

  ```javascript
  import React from "react";
  import { useSelector } from "react-redux";

  export const TodoListItem = (props) => {
    const todo = useSelector((state) => state.todos[props.id]);
    return <div>{todo.text}</div>;
  };
  ```

- **다수의 값** 받아오는 방법들

  - `useSelector()` 호출 하나 당 **하나의 field** 를 불러오도록 여러번 호출

    ```javascript
    const open = useSelector((state) => state.open);
    const importId = useSelector((state) => state.importId);
    const importProgress = useSelector((state) => state.importProgress);
    ```

    여러번 호출해도 re-render 는 **한번만** 일어난다.

    > 하나의 함수형 컴포넌트에서 `useSelect()` 를 여러번 호출할 수도 있다. 각각의 호출은 `Redux store` 에 개별적인 **subscription** 을 생성한다. 하지만, `React update batching behavior` 로 인해 하나의 `dispatching` 은 단 **한번의 re-render** 만 발생시킨다.

  - `shallowEqual` 함수를 사용해 **[shallow check](https://stackoverflow.com/questions/36084515/how-does-shallow-compare-work-in-react)** 을 하도록 한다.

    ```javascript
    import { shallowEqual, useSelector } from "react-redux";

    // later
    const selectedData = useSelector(selectorReturningObject, shallowEqual);
    ```

  - memoized selector 를 지원하는 라이브러리를 사용한다. (예. [reselect](https://github.com/reduxjs/reselect))



### - `useDispatch()`

`Redux store` 에 있는 `dispatch` 함수의 reference 를 `return` 해주는 Hook 이다. `action` 을 `dispatch` 할 수 있다.

```javascript
import React from "react";
import { useDispatch } from "react-redux";

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch();

  return (
    <div>
      <span>{value}</span>
      <button onClick={() => dispatch({ type: "increment-counter" })}>
        Increment counter
      </button>
    </div>
  );
};
```

`dispatch` 함수를 포함한 `callback` 함수를 **자식 컴포넌트에게 넘겨 줄 때**는 [`useCallback()`](https://reactjs.org/docs/hooks-reference.html#usecallback) 을 사용해 momize 해줘야 한다. (`callback` 을 `prop` 으로 넘겨줄 때는 memoize 를 하는 것이 좋다.) 이는 자식 컴포넌트가 **불필요하게 render** 되는 것을 방지한다.

예시:

```javascript
import React, { useCallback } from "react";
import { useDispatch } from "react-redux";

export const CounterComponent = ({ value }) => {
  const dispatch = useDispatch();
  const incrementCounter = useCallback(
    () => dispatch({ type: "increment-counter" }),
    [dispatch]
  );

  return (
    <div>
      <span>{value}</span>
      <MyIncrementButton onIncrement={incrementCounter} />
    </div>
  );
};

export const MyIncrementButton = React.memo(({ onIncrement }) => (
  <button onClick={onIncrement}>Increment counter</button>
));
```



### - `useStore()`

`<Provider>` 컴포넌트에 전달된 `Redux store` 의 reference 를 `return` 해주는 Hook 이다.

보통의 경우 `store` 에 접근하기 위해 `useSelector` 를 쓰면 된다.  
하지만, `store` 에 전달된 `reducer` 를 바꿔야한다던지 특수한 경우에 유용할 수 있다.

```javascript
import React from "react";
import { useStore } from "react-redux";

export const CounterComponent = ({ value }) => {
  const store = useStore();

  // EXAMPLE ONLY! Do not do this in a real app.
  // The component will not automatically update if the store state changes
  return <div>{store.getState()}</div>;
};
```

주석에 설명되어 있듯, **컴포넌트의 자동 update 가 막혀버리**므로 실제 App 에서 사용할 일은 없을 것 같다.



### - Custom Context

`Provider` 에 임의의 Context 를 주입할 수 있는 방법인데 사용하지 않을 것 같아서 링크만 기록해둠

- [공식 문서에 설명된 부분](https://react-redux.js.org/api/hooks#custom-context)
- [공식 문서 `Provider` 부분에도 언급됨](https://react-redux.js.org/api/provider)

## Reference

[React-Redux 라이브러리 공식 문서](https://react-redux.js.org/api/hooks)
