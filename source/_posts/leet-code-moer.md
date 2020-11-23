---
title: leetcode刷题记录_摩尔投票
tags: ['算法']
---
**摩尔投票一般用于查找数组中的主元素，时间复杂度O(n),空间复杂度O(1);**
**众数最基本的定义是出现的次数大于总数一般的数。变量x保存当前数字，count用于计数，循环数组当与x相同，count++，不相同count--，count等于0是，将下一项的值赋给x，循环结束如果数组存在众数，x就是该数组的众数**

### 举例
一个非空整数数组中，一个整数出现的次数大于数组长度的1/2，请找到这个数字
``` bash
const fn = nums => {
  let count = 0, curr = nums[0];
  for(let i = 0; i < nums.length; i++) {
    if (count === 0) {
      curr = nums[i];
      count++;
      continue;
    }
    count = nums[i] === curr ? count + 1 : count - 1;
  }
  return curr;
}
```
leetcode: https://leetcode-cn.com/problems/majority-element/submissions/

**思考：上面的题目中已经明确数组中一定存在众数了，如果改一下题目，如果数组中存在众数，则返回这个数字，如果不存在，则返回false；**
``` bash
let count = 0;
for(let i = 0; i < nums.length; i++) {
  if (nums[i] === curr) {
    count++;
  }
}
if (count > nums.length / 2) return curr;
return false
```

**进阶：请找出非空整数数组中，所有出现次数大于总数1/3的数字**
``` bash
**
* 出现次数大于总数1/3的数字最多是两个，同理用摩尔投票的方法做的话，定义两个变量分别计数，
* 循环结束后再判断剩下的两个数字出现的次数是否大1/3，大于的话就是我们要找的数字
*
let fn = nums => {
  let num1 = nums[0], num2 = nums[0], count1 = 0, count2 = 0;
  for (let i = 0; i < nums.length; i++) {
    if (count1 === 0) {
      num1 = nums[i];
      count1++
      continue;
    }
    if (count2 === 0) {
      num2 = nums[i];
      count2++;
      continue;
    }
    if (num1 === nums[i]) {
      count1++;
      continue;
    }
    if (num2 === nums[i]) {
      count2++;
      continue;
    }
    count1--;
    count2--;
  }
  count1 = count2 = 0;
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] === num1) {
      count1++;
    } else if (nums[i] === num2) {
      count2++;
    }
  }
  const res = [];
  if (count1 > nums.length / 3) res.push(num1);
  if (count2 > nums.length / 3) res.push(num2);
  return res;
}
```
leetcode: https://leetcode-cn.com/problems/majority-element-ii/submissions/