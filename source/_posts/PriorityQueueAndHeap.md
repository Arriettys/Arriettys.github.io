---
title: 优先队列与堆
date: 2020-02-01 10:52:00
tags: Python
categories: 数据结构与算法
mathjax: true
---

# 优先队列与堆

## 什么是优先队列

首先让我们来看一个场景

> 例子：
>
> 医院看病，重症患者往往优先治疗，即使他是后来者

可以看出优先队列相对于普通队列，他是根据元素的优先级进行出队的，与入队顺序无关

也许有人会说把元素进行排序也是可以实现的，道理是没错，如果我们是要选出班级中总成绩最高的同学；但对于医院这种情况，你无法预知今天会有多少个病人和其病症程度，每来一个病人你都重新排序的话就显得麻烦了

<!-- more -->

参考：https://www.liwei.party/2019/01/10/algorithms-and-data-structures/priority-queue/#toc-heading-7

## 优先队列的主要操作

优先队列根据优先级的排序分为，最大优先队列和最小优先队列。而优先队列主要有两大操作，即入队和出对（优先级最高的元素）

我们通常可以最基本的数组这种数据结构来实现优先队列

### 普通数组

可直接加入元素，复杂度为$O(1)$；而出队时需要保证最大，所以需要比较全部元素，所以复杂度为$O(n)$

### 有序数组

当我们插入一个元素时，需要维护数组的元素顺序，时间复杂度为$O(n)$，而出队只需获取头部元素即可，复杂度为$O(1)$

可以看出两种形式的数组作为优先队列，复杂度都无法低于$O(n)$，伟大的计算机科学家平衡了入队和出队这两个操作的时间复杂度，这种数据结构就是堆

## 什么是堆

堆的本质其实是完全二叉树

> 完全二叉树的性质，父结点的值大于（或小于）其子节点，而根节点即为最大值（或最小）

并且完全二叉树可以使用数组实现而无需树结构，这样既可以利用数组元素可以快速访问的特点，又让结点和结点之间形成了“父”与“子”的结构关系。而最大优先队列我们也可称为最大堆，最小堆也是这样

>关系公式：
>
>**大顶堆：arr[i] >= arr[2i+1] && arr[i] >= arr[2i+2]**
>
>**小顶堆：arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]**
>
>父结点：parent[i/2]
>
>ps.以上均为向下取整

![](https://s2.ax1x.com/2019/03/18/Amzm0e.png)

### 排序思想

#### 入队

先将元素加入数组尾端（为什么我们要在数组的末尾添加一个元素呢？可不可以在开头、中间？既然我们使用数组来实现堆，对数组添加一个元素来说，实现复杂度最低的操作就是在数组的末尾添加元素，如若不然，要让数组中一部分的元素逐个后移，**因此在数组的末尾加入元素是最自然的想法**），随后与父结点比较，若大于（或小于）父结点则进行交换，再对父结点进行相同操作，直到其父结点大于它或其为根结点，堆调整结束，这个过程称为“上升”

#### 出队

直接获取数组头部元素，之后调整堆：

根结点取出以后，根节点位置为空，于是我们将当前数组的最后一个元素放到 根节点的位置，这样做是**因为交换和移动的次数最少**；回到数组头部，先从的它的左右结点中选取最大的（或最小的），若比自身大（或小）则交换，再对交换的子节点进行相同操作，反复执行，直到子节点都大于（或小于自身）或没有子节点



堆的入队和出队充分利用的二叉树的二分特性，其复杂度都为$O(log n)$



### 构建一个最大堆

现在让我们来实现一个最大堆

```Python
class MaxHeap:
    def __init__(self, capacity):
        # 存储空间
        self.data = [None for _ in range(capacity+1)]
        # 堆总容量
        self.size = capacity
        # 当前堆中元素个数
        self.count = 0
    # 当前堆是否为空
    def isEmpty(self) -> bool:
        return self.count == 0
    
    # 元素入队
    def push(self, item):
        if self.count + 1 > self.size:
            raise Exception("堆的容量不够了")
        self.count += 1
        self.data[self.count] = item
        self.up(self.count)
    
    # 元素出队
    def poll(self) -> int:
        if self.count < 1:
            raise Exception("堆中没有元素")
        res = self.data[1]
        self.data[1], self.data[self.count] = self.data[self.count], self.data[1]
        self.count -= 1
        self.down(1)
        return res

    # 上升调整
    def up(self, k):
        temp = self.data[k]
        while k > 1 and temp > self.data[k//2]:
            self.data[k//2], self.data[k] = self.data[k], self.data[k//2]
            k //= 2
        self.data[k] = temp
    
    # 下降调整
    def down(self, k):
        temp = self.data[k]
        while 2*k <= self.count:
            Max = 2*k 
            if Max + 1 <= self.count and self.data[Max + 1] > self.data[Max]:
                Max += 1
            if self.data[Max] > temp:
                self.data[Max], self.data[k] = self.data[k], self.data[Max]
                k = Max
            else:
                break
        self.data[k] = temp   
```



测试数据

```Python
if __name__ == '__main__':
    max_heap = MaxHeap(6)
    max_heap.push(3)
    print(max_heap.data[1])
    max_heap.push(5)
    print(max_heap.data[1])
    max_heap.push(1)
    print(max_heap.data[1])
    max_heap.push(8)
    print(max_heap.data[1])
    max_heap.push(7)
    print(max_heap.data[1])
    max_heap.push(12)
    print(max_heap.data[1])
    while not max_heap.isEmpty():
        print('取出', max_heap.poll())
```

打印结果

```
3
5
5
8
8
12
取出 12
取出 8
取出 7
取出 5
取出 3
取出 1
```

现在我们就算完成了最大堆的入队和出队操作，最小堆将比较更改即可获得

### 优先队列的应用场景

总结下来，优先队列比较适合需要动态排序的数据，如上文提到的医院随时会来患者，而患者们的症状程度各有不同的情况，就可以使用优先队列来进行排序；又或者当数据量过于庞大且要相互组合进行比较，而你只需要前第几位或后第几位时，这是也可以使用优先队列减少排序次数

如leedcode 第 719 题：[找到第k小的距离对](https://leetcode-cn.com/problems/find-k-th-smallest-pair-distance/)

