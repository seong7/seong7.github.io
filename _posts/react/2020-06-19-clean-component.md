---
layout: post
title: "Clean Components"
date: 2020-06-19
author: Jason
categories: react
---

### 글을 시작하며..

**'어떻게 해야 더 가독성이 높고 효율적인 코드를 작성할 수 있을까?'**  
요즘 React 프로젝트를 진행하며 가장 많이 고민되는 부분이다. github 에서 임의의 유사 프로젝트 repository 를 검색해보고 몇가지 장점을 따라해보기도 하며 코드의 퀄리티를 높이기 위한 시도를 해보았다.

**'무엇을 따라야 할지 몰라 혼란스러울 때'**  
하지만, 다양한 Practice <sub>(_'관례'_ 또는 _'방식'_ 으로 이해하는 것이 맞을 것 같다.)</sub> 들 속에서 무엇을 따르는 것이 Best 인지 가려내는 것은 초보자인 나에겐 너무 어렵고 혼란스러운 일이었다.

계속해서 좋은 Practice 를 찾아 헤매었고, 누군가 속 시원히 가르쳐주면 좋겠다는 생각이 간절해지기도 했다.

그러던 중, 너무나도 적절한 시기에 [Medium](https://medium.com/) 에서 내가 그토록 원하던 글 하나를 추천해주었다.

<br>

---

> 이 글은 아래 원글을 번역 및 정리한 글입니다.  
> <u>원글 작성자로부터 미리 허가 받았다는 점을 밝힙니다.</u> > **원 글 : [Write clean(er) Components & JSX - _Dana Janoskova_ > ](https://itnext.io/write-clean-er-components-jsx-1e70491baded)**  
> _"단지, 글이 더 많은 사람을 도울 수 있다면 좋겠다."_ 며 흔쾌히 허락한 원글 작성자의 생각이 너무 존경스러웠다. 나도 언젠가 도움을 줄 수 있는 개발자로 성장할 것이다.

---

<br>

### 개요

**React 사용의 어려운 점은 명확하고 가장 좋은 practice 가 없다는 점과 모순된 정보가 많다는 점이다.**
<code>(이 글을 끝까지 읽기로 결심했던 도입부이다. 가려운 곳을 긁어 줄 것 같은 느낌이 들었다.)</code>

그래서 이미 진행 중인 프로젝트에 참여를 하게되면, 전해 받는 코드들 대부분이 많은 버그와 문제점을 갖고 있다.

_그래서 나 <sub>(원글 작성자)</sub> 는 실제 프로젝트에서 직접 경험한 사례들을 바탕으로 **작은 Guide 와 Refactoring solution** 을 쓰기로 결정했다._

### 1. 제약없는 props 와 빈 {} 객체

나는 single-purpose component 철학을 따른다. 즉, 너무 복잡한 100 줄짜리 긴 컴포넌트들을 피하고 모든 것을 가능한 최대한 나누려고 한다.

`<UserCard />` 라는 컴포넌트가 있다고 가정하자. 이 컴포넌트의 유일한 목적은 어떤 user object 를 받아 user data 를 보여주는 것이다.

```jsx
import React from "react";
import PropTypes from "prop-types";

const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
    </ul>
  );
};

UserCard.propTypes = {
  user: PropTypes.object,
};

UserCard.defaultProps = {
  user: {},
};

export default UserCard;
```

이 컴포넌트는 prop (user) 없이는 아무 기능을 하지 못한다. **하지만 아무 prop 을 require 하고 있지 않으며 `Cannot access property 'name' of undefined` error 를 방지하기 위해 default value 로 `{}`를 두고 있다.**

이처럼 `{}`를 default value 로 두게되면 component 는 빈 객체를 가지고 render 된다. 즉, 화면 상의 빈 공간을 보여주는 것이다. **하지만 만약 fallback value 나 skeleton loading 을 제공하지 않는다면, data 없이 `UserCard` 컴포넌트를 렌더할 이유가 없다.**

**무엇이 요구되어야 하고 무엇이 요구되지 않아도 되는가?**

종종 우리는 잠시 코딩을 중단하고 이해를 해야한다. prop 중 무엇이 꼭 필요하고 무엇은 필수가 아닌지 알아두어야 한다. 그리고 **필수 prop 이 없다면 컴포넌트가 render 되지 않도록 해야한다.**

그렇다면, 어떻게 작성되어야 하는가?

### 2. 부모 컴포넌트에서의 Conditioning Render (조건부 렌더)

```jsx
import React, { useState } from "react";
import PropTypes from "prop-types";

export const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
    </ul>
  );
};

UserCard.propTypes = {
  user: PropTypes.object.isRequired,
};

export const UserContainer = () => {
  const [user, setUser] = useState(null);

  // do some apiCall here

  return <div>{user && <UserCard user={user} />}</div>;
};
```

여기서는 `user` 객체를 `null` 로 초기화했다. 이유는 `<UserCard />` 를 쉽게 conditioning render (조건 부 렌더) 할 수 있으며, 그와 동시에 초기 값으로 아무런 객체 reference 를 생성하지 않아 메모리를 아낄 수 있다는 장점 때문이다.

또한 data 가 없을 때 보여줄 spinner 와 메세지를 정하는 것도 더 쉬워진다.

```jsx
export const UserContainer = () => {
  const [user, setUser] = useState(null);

  // do some apiCall here

  return <div>{user ? <UserCard user={user} /> : "No data available"}</div>;
};
```

**반드시 기억해야할 것은 이러한 검사를 해당 컴포넌트에서 보다 부모 컴포넌트에서 하는 것이 훨씬 명확하다는 점이다.** (단지 spinner 와 message 를 보여주기 위해 헛되이 컴포넌트를 렌더하는 경우를 수도 없이 본 것 같다.)

자식 컴포넌트인 `<UserCard />` 는 user data 를 보여주는 책임만을 가지고 있다.

API Call 등을 통해 user 객체를 fetch 하고 무엇을 render 할지 결정하는 것은 `<UserContainer />` 의 책임이 되어야 한다. 그러므로 `<UserContainer />` 가 바로 fallback value 를 보여줄 최적의 장소인 것이다.

### 3. Nesting 을 피하고 대신 Early return 을 선택해라

nesting 은 일반적으로 프로그래밍 언어에서 읽고 수정하는데에 큰 혼란을 야기한다. (JavaScript, HTML, `{}` 의 조합인 JSX 에서도 마찬가지다.)

아마도 아래와 같은 코드를 자주 볼 것이다.

```jsx
const NestedComponent = () => {
  // ...

  return (
    <>
      {!isLoading ? (
        <>
          <h2>Some heading</h2>
          <p>Some description</p>
        </>
      ) : (
        <Spinner />
      )}
    </>
  );
};
```

어떻게 개선할 수 있을까?

```jsx
const NestedComponent = () => {
  // ...

  if (isLoading) return <Spinner />;

  return (
    <>
      <h2>Some heading</h2>
      <p>Some description</p>
    </>
  );
};
```

만약 우리가 Conditioning Render 를 한다면 ('올바른 data가 있는지' / 'page 가 loading 중인지' 등의 조건 확인) , 위와 같이 **early return** 을 선택 할 수 있다.

**이를 통해 nesting 을 피할 수 있고 HTML 과 JavaScript 조건문을 혼란스럽게 섞지 않을 수 있으며, 동료나 비 기술팀 협업자에게도 readable 한 코드를 제공할 수 있다.**

> 경력이 많아질 수록 누구에게나 readable 한 코드를 작성하는 것이 중요하다는 것을 깨닫게 된다. 우리의 코드는 컴퓨터 뿐만 아니라 사람들도 이해할 수 있어야 한다. reviewer 와 junior 동료들이 더 이해하기 쉬운 코드를 작성해야한다. 몇 줄의 코드를 더 쓰는 나의 희생이 나와 동료들의 시간을 장기적으로 아껴준다면, 나는 기꺼이 그 희생을 감수할 것이다.

### 4. JSX 내의 JavaScript 코드는 가능한 최소화 해라

위에서 언급한 것처럼, JSX 는 몇가지 언어가 섞여있어 이해하기 조금 까다로운 면이 있다. Senior 인 나도 다른 사람들이 작성한 JSX 코드를 보면 충분히 readable 하지 않다는 것을 종종 발견한다.

```jsx
const CustomInput1 = ({ onChange }) => {
  return (
    <Input
      onChange={(e) => {
        const newValue = getParsedValue(e.target.value);
        onChange(newValue);
      }}
    />
  );
};
```

event handler 가 모두 JSX 내에 정의되어 있고 `<CustomInput1 />` 의 prop 인 onChange 또한 함께 호출되고 있다. 이 예시 코드는 동작하지만, 몇몇 사람들은 `return()` 문을 이해하는데에 애를 먹을 것이다. 실제 프로그램에서는 훨씬더 많은 element 들과 JS logic 들이 포함된다. 그리고 위의 예시처럼 JSX 내에 nesting 하여 작성하면 코드를 쓰는 입장에선 쉬울지 모르겠다.

```jsx
const CustomInput2 = ({ onChange }) => {
  const handleChange = (e) => {
    const newValue = getParsedValue(e.target.value);
    onChange(newValue);
  };

  return <Input onChange={handleChange} />;
};
```

이 예시는 이 전 예시보다 단 두 줄이 더 길어졌다. 하지만 logic 을 명확히 구분해두었다. JSX `return()` 문에서는 JavaScript logic 을 inline 으로 길게 작성하지 않는 것이 좋다. **JSX `return()` 문에서는 HTML tree 가 어떻게 구조화되어 있는지 명확하게 볼 수 있어야하며, 이는 nesting 이 최소화될때 가장 명확해진다.**

### 5. useCallback 과 useEffect 함께 쓰기

hook 이 나오며 사람들은 functional 컴포넌트를 사용하기 시작했다. 컴포넌트 내에서 data 를 fetch 하기 위한 API call 이 필요해졌고 이를 위해 `useEffect` 라는 lifecycle hook 이 필요해졌다. 최초 버전의 document 에서는 `useEffect` 에 빈 dependency 배열 `[]` 을 넣어주면 hook 이 컴포넌트의 mount 와 un-mount 시 에만 동작한다고 설명되어 있다. 그리고 이 방법은 거의 모든 곳에 사용되었다. 하지만 그 후 `exhaustive-deps (철저한 의존성)` rule 이 나왔다.

```jsx
import React, { useState, useEffect } from "react";

import { fetchUserAction } from "../api/actions.js";

const UserContainer = () => {
  const [user, setUser] = useState(null);

  const handleUserFetch = async () => {
    const result = await fetchUserAction();
    setUser(result);
  };

  useEffect(() => {
    handleUserFetch();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  if (!user) return <p>No data available.</p>;

  return <UserCard data={user} />;
};
```

위의 예시처럼 사람들은 `exhaustive-deps` error 를 comment out 처리하여 그냥 사용했다. 왜냐면 이해가 안됐기 때문이다. 하지만 아직도 이 rule 을 이해하지 못하고 comment out 처리하는 것은 올바른 방법이 아니다.

위와 같은 몇몇의 사람들이 이해하지 못한 것은 바로 **'`handleUserFetch()` method 가 매 render 마다 재생성되고 있다는 점 (또는 컴포넌트가 몇번이나 render 되고 있는지)'** 이다. `useEffect` 내부에서 method를 호출하려면 `useCallback` 이 필요하다. 이를 통해 `handleUserFetch()` method 가 재생성되는 것을 방지할 수 있다 (단, dependency 가 변화될 때는 제외). 그리고 그로인해 infinite loop 발생 없이도 <sub>(? 이해가 안되는 부분이다.)</sub> `handleUserFetch()` 를 `useEffect` hook의 dependency 로 사용할 수 있다.

아래는 수정된 코드이다.

```jsx
import React, { useState, useEffect, useCalllback } from "react";

import { fetchUserAction } from "../api/actions.js";

const UserContainer = () => {
  const [user, setUser] = useState(null);

  const handleUserFetch = useCalllback(async () => {
    const result = await fetchUserAction();
    setUser(result);
  }, []);

  useEffect(() => {
    handleUserFetch();
  }, [handleUserFetch]);

  if (!user) return <p>No data available.</p>;

  return <UserCard data={user} />;
};
```

**만약 원하는 user 의 id 값이 `handleUserFetch()`의 매개변수로 필요하다면 id 또한 dependency 배열에 포함되어야 한다.**

`useEffect` 와 `useCallback` 을 이해하기 시작했다면, `useState` 대신 `useReducer` 를 사용해 보거나 또는 아래와 같이 useState 의 setter 함수에 function 을 매개변수로 넣는 방식을 시도해보는 것도 좋다.

```jsx
const handleUserFetch = useCalllback(async () => {
  const result = await fetchUserAction();
  // setUser(result);
	setUser((prevUser) => {...})
}, []);

```

### 6. 비의존적 로직은 컨포넌트 외부로 보내기

아래와 같은 컴포넌트가 있다고 가정하자.

```jsx
const UserCard = ({ user }) => {
  const getUserRole = () => {
    const { roles } = user;
    if (roles.includes("admin")) return "Admin";
    if (roles.includes("maintainer")) return "Maintainer";
    return "Developer";
  };

  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
      <li>{getUserRole()}</li>
    </ul>
  );
};
```

사람들이 class 형 컴포넌트를 점차 쓰지 않게 되면서 위와 같은 구조의 코드를 많이 볼 수 있게 되었다.

이 전 예시와 같이 여기서 `getUserRole()` method 는 render 시 마다 새로 생성된다. **하지만 여기서는 해당 method를 다른 곳에 dependency 로 제공 (useEffect 또는 자식 컴포넌트의 prop 으로 제공) 하는 것이 아니기 때문에 `useCallback` 을 사용하는 것은 불필요한 낭비이다.**

**여기서 `getUserRole()` 은 컴포넌트 안에 선언되어 재 생성될 필요가 없는 function 임을 알아야 한다.**

```jsx
const getUserRole = (roles) => {
  if (roles.includes("admin")) return "Admin";
  if (roles.includes("maintainer")) return "Maintainer";
  return "Developer";
};

const UserCard = ({ user }) => {
  return (
    <ul>
      <li>{user.name}</li>
      <li>{user.age}</li>
      <li>{user.email}</li>
      <li>{getUserRole(user.roles)}</li>
    </ul>
  );
};
```

이 방법으로 `getUserRole()` 과 같은 function 은 다른 외부 파일에 선언해 import 해서 reusable 하게 쓸 수도 있다.

### 7. inline styling 을 사용하지 마라

inline styling 은 js html css 를 혼합시키고 여러가지 부정적인 결과를 초래한다. 특히 inline css 로 설정한 style 은 외부 css 에서 `!imoprtant` 없이는 수정할 수 없다.

### 8. 올바른 HTML 문법을 사용하라

(웹표준<sub>HTML5</sub> 에 대한 포스트에서 다룰 예정이다.)

### 9. Context overuse 를 피해라

(이 부분은 이해가 잘 안되어 나중에 다시 학습할 예정이다.)

### 마지막 - 글쓴이의 조언

> I have only one advice for you — question your knowledge, your learned approaches and your code. Think and seek whether you can do better. Think outside of the box — forget React and JavaScript, look at the project as a whole and ask yourself what makes the most sense architecture-wise.
>
> **"** 이미 알고 있는 지식, 배운 접근법 그리고 당신의 코드를 의심해라. 당신의 코드를 더 개선할 수 있는지를 생각하고 탐구해라. 'React, JavaScript' 라는 틀에서 벗어나 프로젝트 전체를 보고 가장 현명한 구조가 무엇일지 스스로에게 물어라. **"**
