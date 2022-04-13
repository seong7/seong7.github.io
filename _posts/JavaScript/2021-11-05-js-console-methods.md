---
layout: post
title: "JavaScript console method 활용하기"
date: 2021-11-05
categories: JavaScript
---

Web Frontend 개발 중 debugging 목적으로 사용하는 가장 흔한 방법이 바로 `console.log()` 일 것이다.

하지만, JavaScript 의 `console` 객체에는 `log()` method 외에도 상황에 따라 debugging 을 더 쉽게 만들어줄 method 들이 있다.

### **console.table()**

JavaScript Object 들의 Array 를 테이블 형태로 출력해준다.
설명이 필요 없다. 매우 유용!

![Untitled 1](https://user-images.githubusercontent.com/52827441/163089352-7dee8b76-a0e3-443d-8f29-98d08ffb67b5.png)

### console.trace()

App 의 규모가 커졌을 때, 어떠한 함수가 호출되고 있는지 확인하기 위해 `console.log()` 만을 사용한다면, 어디서 해당 함수가 호출되고 있는지 알 수 없을 것이다. 이럴 때 `console.trace()` 를 사용하면 해당 함수가 호출된 stack trace 를 모두 출력해준다.
실제로 SPA 앱 개발 시, 어떤 함수가 어느 컴포넌트에서 호출되고 있는지 몰라 trace 를 통해 debugging 했던 경험이 있다.

### console.group()

다양한 log 가 console 에 기록되다 보면, 분류하기가 힘들어지는 상황이 올 수 있다. 그럴 때 각각의 log 들을 grouping 할 수 있는 method 이다.

```jsx
console.log("외부 로그 1");
console.group("1번 그룹");
console.log("1번 그룹 내부");
console.group("2번 그룹");
console.log("2번 그룹 내부");
console.warn("2번 그룹 내부 warning");
console.groupEnd();
console.log("다시 1번 그룹");
console.groupEnd();
console.log("다시 외부 로그 2");
```

위의 로그를 실행하면 아래와 같이 출력된다.

![Untitled 2](https://user-images.githubusercontent.com/52827441/163089357-b135cce2-4bdf-43f3-bd0b-31f456337d61.png)

이 외의 다양한 정보들은 [여기 링크](https://news.hada.io/topic?id=5309)를 통해 더 살펴 볼 수 있다.

### Reference

[https://developer.mozilla.org/ko/docs/Web/API/Console](https://developer.mozilla.org/ko/docs/Web/API/Console)

[https://velog.io/@gil0127/console-의-다양한-기능](https://velog.io/@gil0127/console-%EC%9D%98-%EB%8B%A4%EC%96%91%ED%95%9C-%EA%B8%B0%EB%8A%A5)
