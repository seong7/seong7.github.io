---
layout: post
title: "[자료구조] Linked List"
date: 2020-12-16
categories: computer_science
---

# Linked List
### Linked List 의 특징

- 순서를 가진다.
- HashTable 의 collision 에 대한 해결책으로 한 Hash 값 주소에 Linked List 를 저장해 충돌이 발생한 값들을 모두 저장할 수 있다.

<br>

### Linked List 의 종류

- Singly Linked List
- Doubly Linked List
- Circular Linked List

<br>

### 구성요소

- head : 첫 node
- tail : 마지막 node (null 을 pointing 함)
- **pointer** : 객체를 참조하는 링크

  > 어떠한 객체가 참조를 받고 있지 않을 때 메모리에서 삭제하는 것을 `Garbage collection` 이라고 부른다.

- method :
  - append : `tail` 에 새로운 node 추가
  - prepend : `head` 에 새로운 node 추가
  - insert : 중간에 새로운 node 추가

<br>

## Singly Linked List

### Singly Linked List 의 특징

- doubly linked list 보다 memory 를 더 적게 쓴다.
- `prev` 속성이 없어 조작해주지 않아도 되므로 `remove` 나 `insert` 작업이 조금 더 빠르다.
- `reverse search` 또는 `reverse traverse` 를 하지 못한다.
- **memory 가 제한적이고 `search` 가 많이 없으며 `remove`, `insert` 에 집중할 때 사용하기 좋다.**

<br>

### Singly Linked List 구현 코드

reverse() 메소드를 통해 순서를 바꿀 수 있다

```javascript
class Node {
  constructor(value) {
    this.node = {
      value,
      next: null,
    };
    return this.node;
  }
}

class LinkedList {
  constructor(value) {
    this.head = {
      value: value,
      next: null,
    };
    this.tail = this.head;
    this.length = 1;
  }

  append(value) {
    const newNode = new Node(value);
    //   const newNode = {
    //     value: value,
    //     next: null
    //   }
    this.tail.next = newNode;
    this.tail = newNode;
    this.length++;
    return this.printList();
  }

  prepend(value) {
    const newNode = new Node(value);
    //   const newNode = {
    //     value: value,
    //     next: null
    //   }
    newNode.next = this.head;
    this.head = newNode;
    this.length++;
    return this.printList();
  }

  printList() {
    const array = [];
    let currentNode = this.head;
    while (currentNode !== null) {
      array.push(currentNode.value);
      currentNode = currentNode.next;
    }
    return array;
  }

  insert(index, value) {
    //Check for proper parameters;
    if (index >= this.length) {
      console.log("append");
      return this.append(value);
    }

    const newNode = new Node(value);
    //   const newNode = {
    //     value: value,
    //     next: null
    //   }
    const leader = this.traverseToIndex(index - 1); // 하나 전의 노드를 찾아야함
    const holdingPointer = leader.next;
    leader.next = newNode;
    newNode.next = holdingPointer;
    this.length++;
    return this.printList();
  }

  traverseToIndex(index) {
    //Check parameters
    // console.log("index", index);
    let counter = 0;
    let currentNode = this.head;
    while (counter !== index) {
      // console.log(currentNode);
      currentNode = currentNode.next;
      counter++;
    }
    return currentNode;
  }

  remove(index) {
    if (index === 0) {
      const holdingPointer = this.head;
      this.head = this.head.next;
      holdingPointer.next = null;
      this.length--;
      return this.printList();
    }
    if (index >= this.length - 1) {
      // 길이보다 크면 tail 삭제로 구현함
      this.tail = this.traverseToIndex(this.length - 1); // tail 바로 앞
      this.tail.next = null;
      this.length--;
      return this.printList();
    }
    let leader = this.traverseToIndex(index - 1);
    leader.next = leader.next.next;
    this.length--;
    return this.printList();
  }

  /* 중요 !!!  */
  reverse() {
    if (!this.head.next) {
      return this.head;
    }

    let first = this.head;
    let second = this.head.next;

    // 원래의 순서대로 도는 loop
    while (second) {
      // tail 의 경우 second 가 null 이므로 while loop escape 됨
      const temp = second.next;
      second.next = first;
      first = second; // 다음 loop 위해 first 에 다음 node (second) 부여
      second = temp; // 다음 loop 위해 second 에 다음 node (second.next __temp) 부여
    }

    this.tail = this.head; // tail 에 원래의 head 를 부여해줌
    this.head.next = null; // 원래의 head.next 에 null 부여
    this.head = first; // while 문의 종료 시점 즉, 원래 순서의 가장 마지막 node (원래 tail) 를 head 에 부여
    return this.printList();
  }
}

let myLinkedList = new LinkedList(10);
```

<br>

## Doubly Linked List

### Doubly Linked List 의 특징

- previous node 에 link 가 연결되어 있다.
- `reverse traverse` 가 가능해진다. (양방향 search 는 조금 더 효율적인 알고리즘을 짤 수 있게 해준다.)
- **단점**은 메모리가 많이 소비된다. *(의문점 : 메모리에 저장된 데이터의 양은 동일해도 메모리 참조가 많으면 메모리 소비가 많아지는가? 왜?)*

<br>

### Doubly Linked List 구현 코드

```javascript
class Node {
  constructor(value) {
    this.node = {
      value,
      next: null,
      prev: null,
    };
    return this.node;
  }
}

class DoublyLinkedList {
  constructor(value) {
    this.head = new Node(value);
    this.tail = this.head;
    this.length = 1;
  }

  // 마지막 노드 추가
  // O(1) : tail 노드의 해쉬가 저장되어 있는 경우
  // O(n) : tail 노드의 해쉬가 저장되어 있지 않은 경우
  append(value) {
    const newNode = new Node(value);
    this.tail.next = newNode;
    newNode.prev = this.tail;
    this.tail = newNode;
    this.length++;
    return this.printList();
  }

  // 첫번째 노드 추가
  // O(1)
  prepend(value) {
    const newNode = new Node(value);
    newNode.next = this.head;
    this.head.prev = newNode;
    this.head = newNode;
    this.length++;
    return this.printList();
  }

  printList() {
    const array = [];
    let currentNode = this.head;
    for (let i = 0; i < this.length; i++) {
      // console.log(currentNode);
      array.push(currentNode.value);
      currentNode = currentNode.next;
    }
    return array;
  }

  // 원하는 위치에 노드 추가
  // O(n)
  insert(index, value) {
    if (index >= this.length) {
      console.log("append");
      return this.append(value);
    }
    const newNode = new Node(value);
    const leader = this.traverseToIndex(index - 1); // 하나 전의 노드를 찾아야함
    const holdingPointer = leader.next;
    leader.next = newNode;
    newNode.prev = leader;
    newNode.next = holdingPointer;
    holdingPointer.prev = newNode;
    this.length++;
    return this.printList();
  }

  // 탐색
  // 찾고자 하는 index 의 크기에 따라 정순 / 역순을 결정할 수 있어 더 효율적으로 탐색 가능
  // O(n)
  traverseToIndex(index) {
    if (index <= this.length / 2) {
      // 정순
      let counter = 0;
      let currentNode = this.head;
      while (counter < index) {
        currentNode = currentNode.next;
        counter++;
      }
    }
    if (index > this.length / 2) {
      // 역순
      let counter = this.length - 1;
      let currentNode = this.tail;
      while (counter > index) {
        currentNode = currentNode.prev;
        counter--;
      }
    }

    return currentNode;
  }

  // 원하는 위치의 노드를 제거
  // O(n)
  remove(index) {
    if (index === 0) {
      const holdingPointer = this.head;
      this.head = this.head.next;
      this.head.prev = null;
      holdingPointer.next = null; // 사라지는 노드가 가진 외부 참조 모두 제거
      this.length--;
      return this.printList();
    }
    if (index >= this.length - 1) {
      const holdingPointer = this.tail;
      this.tail = holdingPointer.prev;
      holdingPointer.prev = null; // 사라지는 노드가 가진 외부 참조 모두 제거
      this.tail.next = null;
      this.length--;
      return this.printList();
    }
    let leader = this.traverseToIndex(index - 1);
    let holdingPointer = leader.next.next;
    leader.next = holdingPointer;
    holdingPointer.prev = leader;
    this.length--;
    return this.printList();
  }
}

let myLinkedList = new DoublyLinkedList(10);
```

<br>

## Circular Linked List

원형 연결 리스트는 왜 필요할까?

Singly Linked List 의 가장 큰 단점은 (tail 포인터가 없다고 가정 <sub>실제로 알고리즘 문제에서도 tail 포인터가 주어지지 않는 경우가 많다.</sub>), List 의 마지막 노드를 추가하기 위해서  `O(n)` 의 시간 복잡도로 List 를 처음부터 끝가지 traverse (순회) 해야한다는 것이다.

**이를 보완하기 위해서는 Circular Linked List** 가 가장 적합하다.

`head` 포인터 대신 **`tail` 포인터**만 저장하고 있다면, List 의 마지막과 처음에 **상수 시간**으로 노드를 추가할 수 있다.  
게다가 Doubly Linked List 처럼 너무 많은 메모리를 사용하지도 않는다.

```javascript
class Node {
  constuctor(data) {
    this.data = data;
    this.next = null;
  }
}

class CircularLinkedList {
  constructor(data) {
    const node = new Node(data);
    this.tail = node;           // head 없이 tail 만 가진다.
    this.tail.next = this.tail; // 노드가 하나일 때도 순환 시켜준다.
    this.length = 1;
  }

  // 마지막 노드 추가
  // O(1)
  append(data) {
    const node = new Node(data);
    if (this.tail === null) {
      this.tail = node;
      this.tail.next = this.tail;
    } else {
      node.next = this.tail.next; // 첫번째 노드와 연결
      this.tail.next = node;
      this.tail = node;
    }
  }

  // 첫번째 노드 추가
  // O(1)
  prepend(data) {
    const node = new Node(data);
    if(this.tail === null) {
      this.tail = node;
      this.tail.next = this.tail;
    } else {
      node.next = this.tail.next; // 기존 첫번째 노드와 연결
      this.tail.next = node;
      // tail 포인터를 이동시키는 것 빼고는 append 와 동일!
    }
  }

  // 마지막 노드 삭제
  // O(n)  : tail 노드의 전 노드를 찾아야 하므로 traverse 필요
  deleteFromEnd() {
    const target = this.tail;
    if(this.tail === null) {
      return target;
    }
    let prev = this.tail.next;
    while(prev.next !== target) {
        prev = prev.next;
    }
    prev.next = target.next;
    target.next = null;
    this.tail = prev;
    return target;
  }

  // 원하는 위치에 노드 추가
  // O(n)
  insert(index, data) {
    // Singly Linked List 와 동일
  }
}
```