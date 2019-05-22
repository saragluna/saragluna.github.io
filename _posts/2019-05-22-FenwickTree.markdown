---
layout:     post
title:      "Fenwick Tree"
subtitle:   "Binary Indexed Tree"
date:       "2019-05-21 18:00:00"
author:     "Sarag"
header-img: "img/post-bg-2019.jpg"
tags:
    - Algorithms
---



Recently I began to kown this algorithm `Fenwick Tree Algorithm` or `Binary Indexed Tree` while solving this [leetcode problem](https://leetcode.com/problems/range-sum-query-mutable/). But it took me some time to understand this algorithm so I want to write my understanding down.



## Range Sum

### Immutable Array

Given an immutable array `arr` and asked to query it's range sum like `[begin, end]` inclusively multiple times, the intuitive way to solve this problem is to use an array to keep prefix sum, specifically for each index i in such array `sum[i] = arr[0] + arr[1] +...+ arr[i]`. Thus we can get rangeSum by using `sum[j] - sum[i]`. But this only suits the immutable array case, what if we can update the array?

### Mutable Array

#### Brute Force

The easiest way to calculate range sum of mutable array is to do the adding each time, thus the time complexity is `O(mn)` for m times query. 



## Fenwick Tree

The idea behind fenwick tree is that an integer is the sum of appropriate powers of two. So Fenwick uses an array to store accumulative sum, but each index's storing capacity is different according to the index's binary representation.

![pic from Fenwick's paper](/img/in-post/fenwick_tree_array.png)

```js
arr[15] = (1111) = n[ 1] + n[ 2] + ... + n[13] + n[14]] + n[15]
arr[14] = (1110) = n[13] + n[14]
arr[13] = (1101) = n[13]
arr[12] = (1100) = n[ 9] + n[10] + n[11] + n[12] 
arr[11] = (1011) = n[11]
arr[10] = (1010) = n[ 9] + n[10]
arr[ 9] = (1001) = n[ 9]
arr[ 8] = (1000) = n[ 1] + n[ 2] + n[3] + n[4] + n[5] + n[6] + n[7] + n[8]
arr[ 7] = (0111) = n[ 7]
arr[ 6] = (0110) = n[ 6]
arr[ 5] = (0101) = n[ 5]
arr[ 4] = (0100) = n[ 1] + n[ 2] + n[3] + n[4]
arr[ 3] = (0011) = n[ 3]
arr[ 2] = (0010) = n[ 1] + n[ 2]
arr[ 1] = (0001) = n[ 1]
```



The two things to think about are `what to store for each index in the array` and `how to find the prefix sum for each index i`.

- what to store on index **i**

  > - Say i = 5`(0101)`, then `arr[i`] will store the sum of 1 number, specifically `n[5] `
  >
  > - Say i = 6`(0110)`, then `arr[i]` will store the sum of 2 numbers, specifically `n[5] + n[6] `
  >
  > This is because each index of Fenwick Tree array has the capacity to store the sum of  1, 2, 4, 8, …, 2^n numbers of data, and this can be determined by find the **lowbit** of the index.   
  >
  > Take 6 for example, 6 is not power of 2, but when we see 6 in binary 0110, each 1 bit in the binary form represents a power of 2, and the index's capacity is exactly the same as the right most 1 of the binary. 

- how to find prefix sum for index **i**

  > Take 15 for example, if we walk backwardly: 
  >
  > - index 15 stores the sum of 1 number, so we jump 1 distance to index 14 
  > - index 14 stores the sum of 2 numbers, so we add that to sum, and jump 2 distances to 12 
  > - index 12 stores the sum of 4 numbers, so we add that to sum, and jump 4 distances to 8 
  > - index 8 stores the sum of 8 numbers, so we add that to sum, and jump 8 distances to 0 
  > - the end 

![how to determine prefix sum](/img/in-post/fenwick_tree_array_2.png)



### Update

The thing is if we update `arr[i]` by delta, `arr[i] += delta`, we also have to update all the indexes the include i in its capacity. How to determine such indexes?

>If current index's capacity is 1, then go forward 1 more step to update the next index of capacity 2.  
>
>If current index's capacity is 2, then go forward 2 more steps to update the next index of capacity 4.   
>
>if current index's capacity is 4, then go forward 4 more steps to update the next index of capacity 8.  
>
>…  
>
>For each index `x`, go forward `lowbit(x)` more steps to update next index of capacity  `2 * lowbit(x)`

```java
public void update(int pos, int delta) {
  while (pos < arr.length) {
    arr[pos] += delta;
    pos += lowbit(pos);
  }
}
```



### PrefixSum

```java
public void prefixSum(int i) {
  int sum = 0;
  while (i > 0) {
    sum += arr[i];
    i -= lowbit(i);
  }
  return sum;
}
```



### Build

If we have an initial nums array and use an empty array with size N to represents the Fenwick Tree. We have to run N times update to init Fenwick Tree. Thus the time complexity is `O(N * logN)`. We can do the init in `O(N)` time complexity.

```java
// O(N) is calculated by 1/2N + 1/4N + 1/8N... = N
public void build(int[] nums) {
  for (int step = 1; step < nums.lenght; step *= 2) {
    for (int i = 2 * step; i < nums.length; i += 2 * step) {
      arr[i] += arr[i - step];
    }
  }
}
```



### lowbit

> How to compute lowbit of n?
>
> Let n be represented as `xxxxxx1000`, we don't care the high bits ahead of the right most 1 bit.  
>
> Then `~n` will be represented as `yyyyyy0111`, `~n + 1 = yyyyyy1000, x & y = 0`.   
>
> So `n & (~n + 1) = 0000001000`, such is the lowbit we want.
>
> And since `-n` is represented as 2's complement of n equals to `~n + 1`, we use `n & -n` to compute lowbit.  

```java
public int lowbit(x) {
  return x & -x;
}
```



### References

[1 Fenwick Tree Demystified](https://notes.tweakblogs.net/blog/9835/fenwick-trees-demystified.html)

[2 Youtube Video](https://www.youtube.com/watch?v=kPaJfAUwViY)