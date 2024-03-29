---
layout: post
title: "비트 연산 정리"
date: 2020-07-13
categories: data_structures
---

### Integer 를 bit 로 보기

- Integer 는 `4 bytes = 32 bits (32 개의 1 과 0)` 로 이루어짐

- 32 bits 로 표현할 수 있는 수의 크기

  - 2<sup>32</sup> 는 `1000...000` 형태로 0 이 32 개여야한다. <u>(총 33bits)</u>  
    그러므로 32 bits 는 **2<sup>32</sup> - 1** 만큼의 수를 표현할 수 있다.

- 하지만, 정수에는 양의 정수, 음의 정수가 있다.

  - 32 bits 의 첫번째 자리를 `+` 또는 `-` 를 나타내는 부호로 사용하기로 함  
    (1 은 `-` , 0 은 `+`)
  - 즉, 양의 정수에서 사용하는 bit 수가 하나 줄었다.  
    **-> 가장 큰 양의 정수는 2<sup>31</sup> - 1**

  - 음의 정수는 1을 빼지 않는다. (양수에서 0을 이미 표현했으므로)  
    **-> 가장 작은 음의 정수는 - 2<sup>31</sup>** <sub>`10000...00` (`0` 31개)</sub>
    - 참고로 **가장 큰 음의 정수**는 **-1** <sub>`111111....11` (`1` 32개)</sub>

- 이에 따라 Integer 의 범위는 **-2<sup>31</sup> ~ 2<sup>31</sup> - 1** 이된다.  
  (약 -21억... ~ +21억...)



### 비트 연산

- `+ - *` 연산은 10진수와 동일한 방법 사용.

- OR `|` 연산 : 두 비트 중 하나라도 1이면 1

- AND `&` 연산 : 두 비트 모두 1이면 1

- NOT `~` 연산 : 비트 값을 모두 반대로 변환

- XOR `^` 연산 : 두 비트 값이 다르면 1 같으면 0

- Left Shift `<< 수` 연산 :  
  비트 값들을 `수` 만큼 왼쪽 방향으로 이동하고 `0` 을 채워 넣음

- Right Shift

  - **`>> 수`** 연산 (arithmatic right shift) : 부호에 일치하는 수를 채움 (음수는 `1` , 양수는 `0`)
  - **`>>> 수`** 연산 (logical right shift) : 부호와 상관없이 `0` 을 채움



**Shift 연산과 `* 2` 또는 `/ 2` 와 다른 점 :**  
shift 연산으로 32 비트 자리를 벗어난 값들은 소멸되므로 `* 2` 또는 `/ 2` 와 다르다. 정확한 값의 계산을 할 경우는 `* 2` 또는 `/ 2` 사용)



### 비트 연산 응용 1

```
x = 어떤 수

XOR   // 두 비트가 다르면 1 같으면 0
x ^ 0 = x
x ^ 1 = ~ x

AND   // 두 비트 모두 1일때만 1
x & 0 = 0
x & 1 = x
x & x = x

OR    // 두 비트 중 하나라도 1이면 1
x | 0 = x
x | 1 = 1
x | x = x
```



### 비트 연산 응용 2 (coding)

1. **`num` 의 `idx` 번째 수가 `1`인지 `0`인지 판별하는 함수**

   ```javascript
   function test(num, idx) {
     return (num & (1 << idx)) !== 0;
     // 1. 1 << idx 로 idx 번째만 1 인 2진수 생성
     // 2. num 과 AND 연산을 통해 해당 자리만 빼고 모두 0으로 전환
     // 3. 0 인지 검증

     /*
      num = 9
      idx = 3
   
      9 = 1001
      1 << 3 = 1000
   
      1001
      & 1000
      = 1000
   
      1000 !== 0  = true
   */
   }
   ```

2. **`num` 의 `idx` 번째 수를 `1`로 바꿔주는 함수**

   ```javascript
   function test(num, idx) {
     return num | (1 << idx);
     // 1. 1 << idx 로 idx 번째만 1 인 2진수 생성
     // 2. num 과 OR 연산으로 idx 자리만 1 로 전환

     /*
      num = 9
      idx = 2
   
      9 = 1001
      1 << 2 = 100
   
      1001
      | 0100
      = 1101
   */
   }
   ```

3. **`num` 의 `idx` 번째 수를 `0`으로 바꿔주는 함수**

   ```javascript
   function test(num, idx) {
     return num & ~(1 << idx);
     // 1. 1 << idx 로 idx 번째만 1 인 2진수 생성
     // 2. 해당 2진수에 NOT 연산으로 idx 자리만 0  나머지는 1 인 2진수 생성
     // 3. num 과 AND 연산으로 idx 자리만 0 으로 전환

     /*
      num = 9
      idx = 3
   
      9 = 1001
      1 << 3 = 1000
      ~ 1000 = 0111
   
      1001
      & 0111
      = 0001
   */
   }
   ```

4. **`num` 의 `idx` 번째 수부터 왼쪽을 모두 `0` 으로 바꿔주는 함수**

   ```javascript
   function test(num, idx) {
     return num & ((1 << idx) - 1);
     // 1. 1 << idx 로 idx 번째만 1 인 2진수 생성
     // 2. 해당 2진수에 - 1 해서 idx 만 0  나머지는 1 인 2진수 생성
     // 3. num 과 AND 연산으로 idx 왼쪽 자리만 0 으로 전환

     /*
      num = 53
      idx = 3
   
      53 = 110101
      1 << 3 = 1000
      1000 - 1 = 0111     
      * 참고 :  
         *  ~ 001000 = 110111 (not 연산 ) : 해당 자리 제외 모두 변환됨 
         *  001000 - 1 = 000111 (보통 2진수 계산) : 해당 자리 오른쪽으로만 변환됨
   
      110101
      & 000111
      = 000101
   */
   }
   ```

5. **`num` 의 `idx` 번째 수부터 오른쪽을 모두 `0` 으로 바꿔주는 함수**

   ```javascript
   function test(num, idx) {
     return num & (-1 << (idx + 1));
     // 1. - 1 << (idx + 1) 로 전체 1 에 idx 부터 오른쪽으로 0인 2진수 생성
     // 2. num 과 AND 연산으로 idx 포한 오른쪽 자리만

     /*
      num = 53
      idx = 3
   
      53 = 110101
      -1 << (3 + 1) = 1111...1110000  (-1 = 11111...111 (가장 큰 음수))
   
            110101
      & 11...110000
      = 0...0110000
   */
   }
   ```

6. **`num` 의 `idx` 자리 수만 `x` 로 바꿔주는 함수**

   ```javascript
   /**
   * @param {number} num
   * @param {number} idx
   * @param {boolean} x  : set 해줄 값 (true = 1 / false = 0)
   */
   function test(num, idx, x) {
   return (num & ~(1 << idx)) | ((x ? 1: 0) << idx));
   // 1. 문제 3 을 이용해 idx 자리를 0 으로 바꾼 2진수 생성
   // 2. idx 자리만 x 값인 2진수 생성
   // 3. 두 2진수를 OR 연산하면 num 의 idx 자리만 x 값으로 바뀜

   /*
      num = 53
      idx = 3

      53 = 110101
      num & ~(1 << idx) = 110101
      (x ? 1: 0) << idx) = 00x000

      110101
      & 110111
      = 110101 (idx 자리만 0으로 바꾼 2진수) _ 문제 3 참조
      | 00x000 (idx 자리만 x 인 2진수 생성)
      = 11x101
   */
   }
   ```



### 출처

엔지니어대한민국 님의 **[유튜브 영상](https://www.youtube.com/watch?v=yHBYeguDR0A)**
