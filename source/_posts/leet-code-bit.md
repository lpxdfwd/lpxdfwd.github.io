---
title: leetcode刷题记录_位操作
tags: ['算法']
---
在刷leetcode过程中发现二进制的位操作比较常见，因此做个记录

## ^（异或）的使用
### 一样的为真，不一样为假
``` bash
0 ^ 0 = 0; 1 ^ 1 = 0; 1 ^ 0 = 1;
```
### 举例：一个由多个整数组成的数组中，除了一个数字只有一个，其余的两两相同，找出只有一个的数字；

``` bash
/**
* 这道题可以使用到异或的特性，异或数组中的每一项，
* 相同的数字异或为0，最后得到的就是单独的那个数字
*/
const fn = nums => {
  let res = 0;
  for (let i = 0; i < nums.length; i++) {
    res ^= nums[i];
  }
  return res;
}
```

leetcode: https://leetcode-cn.com/problems/single-number/

## >> << 操作
### >> 二进制右移一位（移除二进制的最后一位，在头部补0），一般用于取中位数，向下取整；
### << 二进制左移一位（移除二进制的开头一位，在尾部补0），当前数值*2；

**在js中数字的范围是Math.pow(-2, 31)到Math.pow(2, 31) - 1, 这个范围对应的二进制，在64位的区间内；不管左移还是右移，超过了这个区间的位置都为0；**

### 举例：求n的x次方，x为整数，不可以使用Math.pow方法

``` bash
const fn = (n, x) => {
  if (n === 0) return 0;
  if (x === 0) return 1;
  n = x > 0 ? n : 1 / n;
  let res = 1;
  while(x) {
    if (x & 1 === 1) res *= n;
    n *= n;
    x >>= 1;
  }
  return res;
}
```

leetcode: https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/

## &(与)、｜（或）操作
### & 全为1等于1，否则等于0；
### | 只要有一个为1就等于1，否则等于0；

### 举例：一个由整数组成的数组，除了一个数字只有一个，其余的数字都是三三相同，请找出单独的那个数字；

``` bash
/**
* 这道题使用位运算实现的思路是，将所有数字想像成二进制，统计每个数字相同位上的1的个数，
* 由于只有一个数字是单独的，其余的三三相同，所以每个位上1的个数%3的结果，就是要找的这个数字二进制每个位上的值；
*/
const fn = nums => {
  const arr = [...Array(32)].fill(0);
  for (let i = 0; i < nums.length; i++) {
    let item = nums[i];
    for (let j = 0; j < 32 && item > 0; j++) {
      arr[j] += (item & 1);
      item >>= 1;
    }
  }
  let res = 0;
  for (let i = 0; i < 32; i++) {
    res <<= 1;
    res |= (arr[31 - i] % 3);
  }
  return res;
}
```

leetcode: https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/submissions/

### 总结练习：一个由整数组成的数组，除了2个数字只有一个，其余的数字都是两两相同，请找出单独的那2个数字；

``` bash
/**
* 这道题使用位运算实现的思路是，将所有数字异或，两两相同的数字异或为0，最后得到的值就是这两个数字异或的结果；
* 那么我们如何利用这个结果得到两个数字呢？异或的特性是0 ^ 1 = 0; 我们就可以找到两个数字的二进制位，第一处不相同的地方；
* 1 & 这个数字二进制的最后一位，值为1的位置就是我们要找的位置，如果为0，就将二进制右移一位；直到找到这个位置;
* 循环数组中的每一项，该位置为1的^结果，和为0的^结果，就是要找的这两个数；
*/
const fn = nums => {
  let sum = 0;
  for (let i = 0; i <nums.length; i++) {
    sum ^= nums[i];
  }
  let index = 1;
  while((sum & index) === 0) index <<= 1;

  let a = 0, b = 0;
  for(let i = 0; i < nums.length; i++) {
    if ((nums[i] & index) === 0) {
      a ^= nums[i];
    } else {
      b ^= nums[i]';
    }
  }
  return [a, b];
}
```

leetcode: https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/

**总结：二进制的位操作的性能要优于符号运算，并且在一些情况下可以降低算法的时间复杂度、空间复杂度，所以我觉得学习一下还是有必要的，再此为自己的学习做下记录**
