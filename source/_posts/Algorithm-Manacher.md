---
title: Manacher算法
date: 2019-03-29 08:42:46
tags: Java
categories: 数据结构与算法
---
前几天在力扣做了一道题，查找最长回文子串，按照普通的做法是使用中心拓展算法，以每个字符作为回文子串的中心对称点，每次保存前面求得的回文子串的最大值，最后得到的就是最长回文子串，这种方式的时间复杂度是O(n^2)。这种方式的时间复杂度太高，下面介绍时间复杂度为O(n)的Manacher算法。
<!-- more -->
## 1. Manacher算法中的基础概念
因为回文子串中心字符可以是一个或者两个，这样就存在这奇偶性的问题，处理繁琐，所以这里我们使用一个技巧，具体做法是：在字符串首尾，及各字符间各插入一个字符。

举个例子：s="abbahopxpo"，转换为s_new="#a#b#b#a#h#o#p#x#p#o#"，如此，s 里起初有一个偶回文abba和一个奇回文opxpo，被转换为#a#b#b#a#和#o#p#x#p#o#，长度都转换成了奇数。
### 1.1 回文半径数组radius
回文半径数组radius是用来记录以每个位置的字符为回文中心求出的回文半径长度，如下图所示，对于p1所指的位置radius[6]的回文半径是5，每个位置的回文半径组成的数组就是回文数组，所以#a#c#b#b#c#b#d#s#的回文半径数组为[1, 2, 1, 2, 1, 2, 5, 2, 1, 4, 1, 2, 1, 2, 1, 2, 1]。
![](https://s2.ax1x.com/2019/03/29/A0Zcod.png)

### 1.2 最右回文右边界R
一个位置最右回文右边界指的是这个位置及之前的位置的回文子串，所到达的最右边的地方。比如对于字符串#a#c#b#b#c#b#d#s#，求它的每个位置的过程如下：
![](https://s2.ax1x.com/2019/03/29/A0ZjS0.png)
最开始的时候R=-1，到p=0的位置，回文就是其本身，最右回文右边界R=0;p=1时，有回文串#a#，R=2；p=2时，R=2;P=3时，R=6;p=4时，最右回文右边界还是p=3时的右边界，R=6,依次类推。
ps.复杂度的n即这里的右边界移动次数

### 1.3 最右回文右边界的对称中心C
就是上面提到的最右回文右边界的中心点C，如下图，p=4时，R=6，C=3
![](https://s2.ax1x.com/2019/03/29/A0eu0e.png)

## 2. Manacher算法的流程
首先大的方面分为两种情况：
**第一种情况：下一个要移动的位置在最右回文右边界R的右边**
比如在最开始时，R=-1,p的下一个移动的位置为p=0，p=0在R=-1的右边；p=0时，此时的R=0，p的下一个移动位置为p=1，也在R=0的右边。
在这种情况下，采用普遍的解法，将移动的位置为对称中心，向两边扩，同时更新回文半径数组，最右回文右边界R和最右回文右边界的对称中心C。
**第二种情况：下一个要移动的位置就是最右回文右边界R或是在R的左边**
在这种情况下又分为三种：
1、下一个要移动的位置p1在最右回文右边界R左边，且cL<pL
p2是p1以C为对称中心的对称点；

pL是以p2为对称中心的回文子串的左边界;

cL是以C为对称中心的回文子串的左边界。

这种情况下p1的回文半径就是p2的回文半径radius[p2]。
![](https://s2.ax1x.com/2019/03/29/A0eTc6.png)
2、下一个要移动的位置票p1在最右回文右边界R的左边，且cL<pL
p2是p1以C为对称中心的对称点；

pL是以p2为对称中心的回文子串的左边界；

cL是以C为对称中心的回文子串的左边界。

这种情况下p1的回文半径就是p1到R的距离R-p1+1。
![](https://s2.ax1x.com/2019/03/29/A0mQDU.png)
3、下一个要移动的位置票p1不在最右回文右边界R的左边，且cL=pL
p2是p1以C为对称中心的对称点；

pL是以p2为对称中心的回文子串的左边界；

cL是以C为对称中心的回文子串的左边界。

这种情况下p1的回文半径就还要继续往外扩，但是只需要从R之后往外扩就可以了，扩了之后更新R和C。
![](https://s2.ax1x.com/2019/03/29/A0mwDO.png)
## 3. Manacher时间复杂度分析
从上面的分析中，可以看出，第二种情况的1，2的求某个位置的回文半径的时间复杂度是O(1)，对于第一种情况和第二种情况的3，R是不断的向外扩的，不会往回退，而且寻找回文半径时，R之内的位置是不是进行判断的，所以对整个字符串而且，R的移动是从字符串的起点移动到终点，时间复杂度是O(n),所以整个manacher的时间复杂度是O(n)。

## 4. 代码实现
```
class Solution 
{
    public String longestPalindrome(String s) 
    {
        return manacher(s);
    }
    
    public static char[] manacherString(String str)
    {
        StringBuilder sb = new StringBuilder();
        for (int i=0;i<str.length();i++)
        {
            sb.append("#");
            sb.append(str.charAt(i));
        }
        sb.append("#");
        return sb.toString().toCharArray();
    }
    public static String manacher(String str)
    {
        if (str==null || str.length()<1) return "";
        char[] charArr = manacherString(str);
        int C = -1;
        int R = -1;
        int maxRadius = -1;
        int maxC = -1;
        int[] radius = new int[charArr.length];
        for (int i=0;i<radius.length;i++)
        {
            radius[i] = R>i?Math.min(radius[2*C-i],R-i+1):1;
            while (i+radius[i]<radius.length && i-radius[i]>-1)
            {
                if (charArr[i-radius[i]] == charArr[i+radius[i]]) 
                {
                    radius[i]++;
                }
                else 
                {
                    break;
                }
            }
            if (i+radius[i]-1>R)
            {
                R = i+radius[i]-1;
                C = i;
            }
            if (radius[i]>maxRadius)
            {
                maxRadius = radius[i];
                maxC = i;
            }
        }
        StringBuilder sb = new StringBuilder();
        char a = '#';
        for (int i=maxC-maxRadius+1;i<=maxC+maxRadius-1;i++)
        {
            if (a!=charArr[i]) sb.append(charArr[i]);
        }
        return sb.toString();
    }
}
```
以上代码通过力扣测试

参考自：
https://www.jianshu.com/p/116aa58b7d81

