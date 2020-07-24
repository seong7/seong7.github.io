---
layout: post
title: "Leetcode [Medium] 260 - Single Number III"
date: 2020-07-23
author: Jason
categories: algorithms
---

# LeetCode 260

## Single Number III

> [문제 출처](https://leetcode.com/problems/single-number-iii/)

### 문제

```
Given an array of numbers nums, in which exactly two elements appear only once and all the other elements appear exactly twice. Find the two elements that appear only once.

Example:

Input:  [1,2,1,3,2,5]
Output: [3,5]
Note:

The order of the result is not important. So in the above example, [5, 3] is also correct.
Your algorithm should run in linear runtime complexity. Could you implement it using only constant space complexity?
```

### 나의 코드

```javascript
/**
 * @param {number[]} nums
 * @return {number[]}
 */
var singleNumber = function (nums) {
  let once = {};
  let i = nums.length - 1;
  while (i >= 0) {
    i--;
    const cur = nums.pop();
    if (once[cur]) {
      delete once[cur];
    } else if (!once[cur]) {
      once[cur] = true;
    }
  }

  return Object.keys(once);
};
```

### 참고 코드

[링크](<https://leetcode.com/problems/single-number-iii/discuss/501747/Concise-JavaScript-solution-using-XOR-O(1)-space>)

```javascript
var singleNumber = function (nums) {
  var xor = nums.reduce((acc, cur) => acc ^ cur, 0);
  var uniqBitPos = xor.toString(2).length - 1;
  var xor2 = nums
    .filter((num) => ((num >> uniqBitPos) & 1) == 0)
    .reduce((acc, cur) => acc ^ cur, 0);
  return [xor2, xor2 ^ xor];
};
```

### 배운점
