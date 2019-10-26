---
title: 素数筛法
date: 2019-10-24 20:44:27
tags: Java
categories: 数据结构与算法
---

在力扣碰到的题，判断一个数是否为素数，之前也做过类似的题目，这回总结一下素数筛法的知识点

<!-- more -->

素数的定义：除了1和它本身之外，不能被其他整数整除

## 定义法

根据定义，枚举2~n-1个整数，如果不能整除n，则n为素数

~~~java
boolean isPrime(int n)
{
    if (n == 1)
        return false;
    if (int i = 2; i < n; i++)
        if (n % i == 0)
            return false;
    return true;
}
~~~

优化版本：从2开始枚举到不大于sqrt(n)的所有正整数，假设n存在大于或等于sqrt(n)的因子y，则z=n/y必同时为n的因子，且其值小于或等于sqrt(n)，例如12
$$
... \\
3 * 4 = 12 \\
\sqrt{12} * \sqrt{12} = 12 \\
4 * 3 = 12
$$

~~~java
boolean isPrime(int n)
{
    if (n == 1)
        return false;
    for (int i = 2; i * i <= n; i++)
        if (n % i == 0) return false;
    return true;
}
~~~

但如果问题升级到：给定一个正整数n，问n以内有多少个素数，该方法就力不从心了

## 埃氏筛

埃氏筛的定义是枚举n内所有素数，并将其倍数剔除。先用2去，把2留下，其倍数剔除，2筛完后，剩余最小的数一定是素数，因为其前面小于它的数都不是它的合数，如3，不断重复下去，直到n，这个过程同时需要一个长度为n的空间记录

![](https://pic.leetcode-cn.com/23d348bef930ca4bb73f749500f664ccffc5e41467aac0ba9787025392ca207b-1.gif)

~~~java
boolean isPrime(int n)
{
    boolean[] isPrime = new boolean[n];
    Arrays.fill(isPrime, true);
 	for (int i = 2; i*i < n; i++)
        if (isPrime[i])
            for (int j =2*i; j < n; j += i)
                isPrime[j] = false;
    int count = 0;
    for (int i = 2;i < n; i++)
        if (isPrime[i])
            count++;
    return count;
}
~~~

#### 一些优化：

内层循环

~~~java
for (int j = 2 * i; j < n; j +=i)
    isPrime[j] = false
~~~

这个过程将i的倍数都标记为false，但仍然有计算亢余

如i = 4，标记 4 * 2 = 8时，8这个数字已经被i= 2 的2 × 4所标记，所以我们可以从i的平方开始

~~~Java
int countPrimes(int n)
{
    boolean[] isPrime = new boolean[n];
    Arrays.fill(isPrime, true);
 	for (int i = 2; i*i< n; i++)
        if (isPrime[i])
            for (int j =i*i; j < n; j += i)
                isPrime[j] = false;
    int count = 0;
    for (int i = 2;i < n; i++)
        if (isPrime[i])
            count++;
    return count;
}
~~~

## 欧拉筛

### 算法思路

对于每一个数（无论质数合数）x，筛掉所有小于x**最小质因子**的质数，再乘以x的数。比如对于77,它分解质因数是7*11，那么筛掉所有小于7的质数×77，如筛掉2×77、3× 77、5×77

### 算法证明

#### 没有重复筛

我们假设一个合数可以分解为p1×p2×p3，并且满足p1<p2<p3的约束，如154，其可被筛的组合有：2和7×11、7和2×11、11和2×7，而欧拉筛只会选择小于右边因子中最小质因子的质数，所以上述组合只有2和7×11会被循环到（下面代码中 //if (i%prime[j] == 0) break// 语句会限制质数只会取到因子的最小质因子），



```java
int countPrimes(int n)
{
    int result = 0;
    boolean[] flag = new boolean[n];
    int[] prime = new int[n];
    for (int i=2;i<n;i++)
    {
        if (!flag[i])
            prime[result++] = i;
        int j = 0;
        while (j<result && i*prime[j] < n)
        {
            flag[i*prime[j]] = true;
            if (i%prime[j] == 0) break;
     		j++;
        }
    }
    return result;
}
```
