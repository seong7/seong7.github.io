---
layout: post
title: "JS 비동기 문법에 대한 새로운 접근"
date: 2020-04-10
categories: JavaScript
---

Javascript 는 다양한 방법의 비동기처리 문법을 제공합니다.
정리가 필요하여 공부를 하다가, 각 문법들의 "관계"와 "발전 방향"에 대해 접근한 좋은 설명을 들을 기회가 있어 배운 내용과 저의 생각을 정리해보려 합니다.

> 이 글은 각 문법에 대한 자세한 설명은 다루지 않습니다.

---

우선, Javascript 비동기처리 문법의 발전은 크게 아래 두 가지를 목표로 이루어지고 있다고 생각합니다.

- 비동기 코드의 **동작 순서 제어**
- 비동기 코드의 **가독성 향상**

아래 예시들을 통해 더 자세히 다뤄보겠습니다.

## 가장 기본적인 비동기 코드

```javascript
function asyncWork() {
  setTimeout(() => {
    console.log("첫번째");
  }, 3000 * Math.random());

  setTimeout(() => {
    console.log("두번째");
  }, 3000 * Math.random());

  setTimeout(() => {
    console.log("세번째");
  }, 3000 * Math.random());
}
asyncWork();
```

**이 코드의 문제점은 코드의 진행 순서를 제어할 수 없다는 점입니다.**

얼핏보면, 순서대로 나열된 setTimeout() 함수들이 각각 최대 3초의 random 시간 후 차례대로 동작할 것 같지만,
_실제로 세 함수는 거의 동시에 시작한다고 볼 수 있습니다._

**그 이유는 비동기 함수들이 동기적 순서로 나열되어 있기 때문입니다.**

동기적 진행에 따라 setTimeout() 을 차례대로 실행시키지만, 동기적 진행은 background 에서 개별적으로 실행되는 비동기 코드의 존재를 인지하지 못합니다.
즉, 굉장히 짧은 시간만에 다음 동기적 순서의 코드를 실행시키죠.

**그러므로, 실제 출력은**
**Random 순서로 문자열 '첫번째', '두번째', '세번째' 가 출력됩니다.**

## 순서 제어 문제를 해결한 코드

```javascript
function asyncWork() {
  setTimeout(() => {
    console.log("첫번째");
    setTimeout(() => {
      console.log("두번째");
      setTimeout(() => {
        console.log("세번째");
      }, 3000 * Math.random());
    }, 3000 * Math.random());
  }, 3000 * Math.random());
}
asyncWork();
```

**이 코드의 실제 출력은**
**문자열 '첫번째', '두번째', '세번째' 가 순서대로 출력됩니다.**

각각의 setTimeout() 함수는 먼저 실행될 setTimeout() 의 callback 함수에서 호출되고 있습니다.

각 callback 함수 내에서는 동기적 순서로 console.log("순서 표시") 와 다음 setTimeout() 을 호출해주면서 순서 제어 문제를 해결했습니다.

**하지만, 이 코드의 문제점은 가독성이 낮다는 것입니다.**
만약 서버와 다수의 비동기 통신이 필요한 함수라면, 단일 통신을 하나하나 호출할 수 밖에 없는 위 코드에서 문제는 더욱 심각해지겠죠.
이러한 코드를 **Callback Hell** 이라고도 부릅니다.

## 가독성 문제를 완화한 코드

```javascript
async function asyncWork() {
  try {
    // 첫번째
    let result = await new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log("첫번째");
        resolve(" result 에 저장되는 값");
      }, 3000 * Math.random());
    });

    console.log("동기적 코드 하나를 삽입해보자.");

    // 두번째
    await new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log("두번째" + result);
        reject("catch 의 error 에 저장되는 값"); // catch 로 이동
      }, 3000 * Math.random());
    });

    // 세번째
    // 위에서 reject 되어 실행되지 않음
    await new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log("세번째");
      }, 3000 * Math.random());
    });
  } catch (error) {
    console.log(error); // "catch 의 error 에 저장되는 값" 출력되는 곳
  } finally {
    console.log("finally 는 무조건 실행");
  }
}
asyncWork();

// 출력 :
// 첫번째
// 동기적 코드 하나를 삽입해보자.
// 두번째 'result 에 저장되는 값'
// catch 의 error 에 저장되는 값
// finally 는 무조건 실행
```

**위 코드는 Promise 의 사용이 미숙해 여전히 낮은 가독성 문제를 보이고 있습니다.**
하지만, 더 이상 비동기 코드들이 Callback Hell 로 엮여 있지 않고 동기적 순서(위 -> 아래) 로 나열되어 있음을 확인할 수 있습니다.

여기서 짚고 갈 수 있는 중요한 개념이 있습니다.

**await 는 "async function 내에 한해서" 동기적 순서의 코드 진행을 멈추고 비동기 코드가 종료될 때까지 기다린다.**

위 코드 중간에 삽입된 console.log() 는 동기적 코드임에도 비동기 코드들의 존재를 인지하고 종료를 기다린 후 실행됨을 확인할 수 있습니다.
**즉, await 가 동기적 순서를 제어한다는 의미입니다.**

이제 async / await , Promise 를 이용하면 비동기 코드도 동기적 순서로 평범하게 나열할 수 있음을 확인했습니다.
**활용해서 가독성을 크게 향상시킬 수 있겠죠. (다음 코드에서 확인)**

> 주제와는 관련이 적지만,
> Promise 는 매개변수인 resolve 와 reject 중 반드시 하나를 구현부에서 호출해야합니다. 이는 resolve 또는 reject 함수가 해당 Promise 의 상태를 pending (대기) 에서 **fulfilled** (이행) 또는 **rejected** (거부) 로 변경 시키기 때문입니다.
> await 는 Promise 의 상태를 인식하여 성공 또는 실패 여부를 판단합니다.
>
> _만약 resolve, reject 중 아무 것도 호출되지 않으면 해당 Promise 는 pending 상태로 남게되고 다음 코드 (.then() 과 동일) 는 실행되지 않습니다._

## 가독성을 더 향상시킨 코드 (마지막)

```javascript
// 공통된 비동기 작업을 Promise 하나로 묶어줌
function make_promise(number) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log(number);
      resolve(number * 10);
    }, 3000 * Math.random());
  });
}

// Promise consume(사용)은 async 함수 내에서 함
async function asyncWork() {
  try {
    result = await make_promise(1); // number 값
    result = await make_promise(result);
    result = await make_promise(result);
    result = await make_promise(result);
  } catch (error) {
    console.log(error);
  } finally {
    console.log("무조건 실행");
  }
}
asyncWork();

// 출력 :
// 1
// 10
// 100
// 1000
// 무조건 실행
```

드디어 마지막 코드입니다.

제가 글의 처음에 언급했던 주제에 맞춰 풀어보겠습니다.

1. **동작 순서 제어**

   - **async function** 과 **await** 를 이용했으므로 순서를 제어할 수 있습니다.

2. **가독성 향상**
   - **동일한 동작을 하는 비동기 코드들을 하나의 Promise 에 묶어주어** 가독성을 향상시켰습니다. 조금 더 심화하여 매개변수를 받아 처리할 수 있도록 function 을 선언해 Promise 를 return 해주었습니다.
   - **async / await** 로 비동기 코드들을 동기적 순서로 나열한 것이 가독성을 향상시키기도 했습니다.

---
