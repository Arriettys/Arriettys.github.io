---
title: 常用的八种排序算法和代码实现
date: 2019-03-17 17:13:55
tags: Java
categories: 数据结构与算法
---
![](https://s2.ax1x.com/2019/03/17/AeRNf1.png)
<!--more-->
## 直接插入排序
我们经常会到这样一类排序问题：把新的数据插入到已经排好的数据列中。将第一个数和第二个数排序，然后构成一个有序序列将第三个数插入进去，构成一个新的有序序列。对第四个数、第五个数……直到最后一个数，重复第二步。

直接插入排序（Straight Insertion Sorting）的基本思想：在要排序的一组数中，假设前面(n-1) [n>=2] 个数已经是排好顺序的，现在要把第n个数插到前面的有序数中，使得这n个数也是排好顺序的。如此反复循环，直到全部排好顺序。

### 代码实现
```
public void insertSort(int[] li)
{
	int len = li.length;
	for (int i=1;i<len;i++)
	{
		int insertNum = li[i];
		int last = i - 1;
		while last>=0 && insertNum<li[last])
		{
			li[last + 1] = li[last];
			last--;
		}
		li[last+1] = insertNum;
	}
}
```

## 希尔排序
希尔排序是希尔（Donald Shell）于1959年提出的一种排序算法。希尔排序也是一种插入排序，它是简单插入排序经过改进之后的一个更高效的版本，也称为缩小增量排序，同时该算法是冲破O(n2）的第一批算法之一。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又叫缩小增量排序。

希尔排序是把记录按下表的一定增量分组，对每组使用直接插入排序算法排序；随着增量逐渐减少，每组包含的关键词越来越多，当增量减至1时，整个文件恰被分成一组，算法便终止。

### 算法描述
我们来看下希尔排序的基本步骤，在此我们选择增量gap=length/2，缩小增量继续以gap = gap/2的方式，这种增量选择我们可以用一个序列来表示，{n/2,(n/2)/2...1}，称为增量序列。希尔排序的增量序列的选择与证明是个数学难题，我们选择的这个增量序列是比较常用的，也是希尔建议的增量，称为希尔增量，但其实这个增量序列不是最优的。此处我们做示例使用希尔增量。

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：

选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
按增量序列个数k，对序列进行k 趟排序；
每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。
### 过程演示
![](https://s2.ax1x.com/2019/03/17/AeoD29.png)

### 代码实现
```
public static void shellSort(int[] li)
{
	int len = li.length;
	int gap = len/2;
	while (gap>0)
	{
		for (int i=gap;i<len;i++)
		{
			int insertNum = li[i];
			int last = i - gap;
			while (last>=0 && li[last]>insertNum)
			{
				li[last+gap] = li[last];
				last -= gap;
			}
			li[last+gap] = insertNum;
		}
		gap /= 2;
	}
}
```

## 冒泡排序
冒泡排序是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越大(或小)的元素会经由交换慢慢“浮”到数列的顶端。
### 代码实现
```
public static void bubbleSort(int[] li)
{
	int len = li.length;
	for (int i=0;i<len-1;i++)
	{
		for (int j=0;j<len-1-i;j++)
		{
			if (li[j]>li[j+1])
			{
				int temp = li[j];
				li[j] = li[j+1];
				li[j+1] = temp;
			}
		}
	}
}
```

## 选择排序
表现最稳定的排序算法之一，因为无论什么数据进去都是O(n2)的时间复杂度，所以用到它的时候，数据规模越小越好。唯一的好处可能就是不占用额外的内存空间了吧。理论上讲，选择排序可能也是平时排序一般人想到的最多的排序方法了吧。

选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。 
### 代码实现
```
public static void selectSort(int[] li)
{
	int len = li.length;
	for (int i=0;i<len;i++)
	{
		int minIndex = i;
		for (int j=i+1;j<len;j++)
		{
			if (li[j]<li[minIndex])
			{
				minIndex = j;
			}
		}
		int temp = li[i];
		li[i] = li[minIndex];
		li[minIndex] = temp;
	}
}
```

## 归并排序
归并排序是建立在归并操作上的一种有效的排序算法。该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。归并排序是一种稳定的排序方法。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为2-路归并。 

+ 把长度为n的输入序列分成两个长度为n/2的子序列；
+ 对这两个子序列分别采用归并排序；
+ 将两个排序好的子序列合并成一个最终的排序序列。

![](https://s2.ax1x.com/2019/03/18/AmAYs1.gif)
### 代码实现
	public static int[] margeSort(int[] li)
	{
		int len = li.length;
		if (len<2) return li;
		int mid = len/2;
		int[] left = Arrays.copyOfRange(li, 0, mid);
		int[] right = Arrays.copyOfRange(li, mid, len);
		return marge(margeSort(left), margeSort(right));
		
	}
	public static int[] marge(int[] left, int[] right)
	{
		int[] result = new int[left.length + right.length];
		for (int index=0,i=0,j=0;index<result.length;index++)
		{
			if (i>=left.length)
			{
				result[index] = right[j++];
			} else if(j>=right.length)
			{
				result[index] = left[i++];
			} else if (left[i]>right[j])
			{
				result[index] = right[j++];
			} else if (left[i]<right[j])
			{
				result[index] = left[i++];
			}
		}
		return result;
	}

## 快速排序
快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。
![](https://s2.ax1x.com/2019/03/18/AmESSJ.gif)
### 代码实现
```
	public static void quickSort(int[] li, int start, int end)
	{
		int len = li.length;
		if (len<0 || start>=end) return;
		int mid = (int) (start + Math.random()*(end - start + 1));
		int sortIndex = start - 1;
		swap(li, mid, end);
		for (int i=start;i<=end;i++)
		{
			if (li[i]<=li[end])
			{
				sortIndex++;
				if (i>sortIndex)
				{
					swap(li,i,sortIndex);
				}
			}
		}
		if (start<=sortIndex && sortIndex<=end)
		{
			quickSort(li, start, sortIndex-1);
			quickSort(li, sortIndex+1, end);
		}
	}
	
	public static void swap(int[] li, int i, int j)
	{
		int temp = li[i];
		li[i] = li[j];
		li[j] = temp;
	}
```
## 堆排序
堆排序比较难理解，下面描述转载自：https://www.cnblogs.com/chengxiao/p/6129630.html

### 预备知识
堆排序是利用堆这种数据结构而设计的一种排序算法，堆排序是一种选择排序，它的最坏，最好，平均时间复杂度均为O(nlogn)，它也是不稳定排序。首先简单了解下堆结构。

#### 堆
　　堆是具有以下性质的完全二叉树：每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆；或者每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆。如下图：
　　
![](https://s2.ax1x.com/2019/03/18/Am4noD.png)
同时，我们对堆中的结点按层进行编号，将这种逻辑结构映射到数组中就是下面这个样子

![](https://s2.ax1x.com/2019/03/18/Am43QI.png)
该数组从逻辑上讲就是一个堆结构，我们用简单的公式来描述一下堆的定义就是：

**大顶堆：arr[i] >= arr[2i+1] && arr[i] >= arr[2i+2]  **

**小顶堆：arr[i] <= arr[2i+1] && arr[i] <= arr[2i+2]  **

ok，了解了这些定义。接下来，我们来看看堆排序的基本思想及基本步骤：
### 堆排序基本思想及步骤
堆排序的基本思想是：将待排序序列构造成一个大顶堆，此时，整个序列的最大值就是堆顶的根节点。将其与末尾元素进行交换，此时末尾就为最大值。然后将剩余n-1个元素重新构造成一个堆，这样会得到n个元素的次小值。如此反复执行，便能得到一个有序序列了

**步骤一 构造初始堆。将给定无序序列构造成一个大顶堆（一般升序采用大顶堆，降序采用小顶堆)。**
1.假设给定无序序列结构如下
![](https://s2.ax1x.com/2019/03/18/Amzm0e.png)
2.此时我们从最后一个非叶子结点开始（叶结点自然不用调整，第一个非叶子结点 arr.length/2-1=5/2-1=1，也就是下面的6结点），从左至右，从下至上进行调整。
![](https://s2.ax1x.com/2019/03/18/Amz86f.png)
3.找到第二个非叶节点4，由于[4,9,8]中9元素最大，4和9交换。
![](https://s2.ax1x.com/2019/03/18/Amzwhn.png)
4.这时，交换导致了子根[4,5,6]结构混乱，继续调整，[4,5,6]中6最大，交换4和6。
![](https://s2.ax1x.com/2019/03/18/AmzsXT.png)
此时，我们就将一个无需序列构造成了一个大顶堆。
**步骤二 将堆顶元素与末尾元素进行交换，使末尾元素最大。然后继续调整堆，再将堆顶元素与末尾元素交换，得到第二大元素。如此反复进行交换、重建、交换。**
a.将堆顶元素9和末尾元素4进行交换
![](https://s2.ax1x.com/2019/03/18/AmzfhR.png)
b.重新调整结构，使其继续满足堆定义
![](https://s2.ax1x.com/2019/03/18/Amzg74.png)
c.再将堆顶元素8与末尾元素5进行交换，得到第二大元素8.
![](https://s2.ax1x.com/2019/03/18/Amz7nO.png)
后续过程，继续进行调整，交换，如此反复进行，最终使得整个序列有序
![](https://s2.ax1x.com/2019/03/18/AnS1UJ.png)
再简单总结下堆排序的基本思路：
**a.将无需序列构建成一个堆，根据升序降序需求选择大顶堆或小顶堆;**
**b.将堆顶元素与末尾元素交换，将最大元素"沉"到数组末端;**
**c.重新调整结构，使其满足堆定义，然后继续交换堆顶元素与当前末尾元素，反复执行调整+交换步骤，直到整个序列有序**
### 代码实现
```
	public static void heapSort(int[] li)
	{
		int len = li.length;
		if (len<=0) return;
		for (int i=len/2-1;i>=0;i--) adjustHeap(li,i,len);
		for (int i=len-1;i>0;i--)
		{
			swap(li,0,i);
			adjustHeap(li,0,i);
		}
		
	}
	public static void adjustHeap(int[] li, int rootIndex, int len)
	{
		int root = li[rootIndex];
		for (int k=2*rootIndex+1;k<len;k=2*k+1)
		{
			if (k+1<len && li[k]<li[k+1]) k++;
			if (root<li[k]) 
			{
				li[rootIndex] = li[k];
				rootIndex = k;
			} else
			{
				break;
			}
		}
		li[rootIndex] = root;
	}
	
	public static void swap(int[] li, int i, int j)
	{
		int temp = li[i];
		li[i] = li[j];
		li[j] = temp;
	}
```
## 计数排序
计数排序的核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。 作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

计数排序(Counting sort)是一种稳定的排序算法。计数排序使用一个额外的数组C，其中第i个元素是待排序数组A中值等于i的元素的个数。然后根据数组C来将A中的元素排到正确的位置。它只能对整数进行排序。

### 动画演示
![](https://s2.ax1x.com/2019/03/18/AnitBV.gif)
### 代码实现
```
	public static void countSort(int[] li)
	{
		int len = li.length;
		if (len==0) return;
		int min = li[0], max = li[0];
		for (int i=1;i<len;i++)
		{
			if (min>li[i]) min = li[i];
			if (max<li[i]) max = li[i];
		}
		int[] bucket = new int[max - min + 1];
		Arrays.fill(bucket, 0);
		for (int i=0;i<len;i++) bucket[li[i] - min]++;
		int index = 0;
		for (int i=0;i<bucket.length;i++)
		{
			while (bucket[i]!=0)
			{
				li[index++] = i + min;
				bucket[i]--;
			}
		}
	}
```
## 桶排序
桶排序是计数排序的升级版。它利用了函数的映射关系，高效与否的关键就在于这个映射函数的确定。

桶排序 (Bucket sort)的工作的原理：假设输入数据服从均匀分布，将数据分到有限数量的桶里，每个桶再分别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排
### 算法描述

+ 人为设置一个BucketSize，作为每个桶所能放置多少个不同数值（例如当BucketSize==5时，该桶可以存放｛1,2,3,4,5｝这几种数字，但是容量不限，即可以存放100个3）；
+ 遍历输入数据，并且把数据一个一个放到对应的桶里去；
+ 对每个不是空的桶进行排序，可以使用其它排序方法，也可以递归使用桶排序；
+ 从不是空的桶里把排好序的数据拼接起来。 
### 代码实现
```
/**
     * 桶排序
     * 
     * @param array
     * @param bucketSize
     * @return
     */
    public static ArrayList<Integer> BucketSort(ArrayList<Integer> array, int bucketSize) 
    {
        if (array == null || array.size() < 2)
            return array;
        int max = array.get(0), min = array.get(0);
        // 找到最大值最小值
        for (int i = 0; i < array.size(); i++) 
        {
            if (array.get(i) > max)
                max = array.get(i);
            if (array.get(i) < min)
                min = array.get(i);
        }
        int bucketCount = (max - min) / bucketSize + 1;
        ArrayList<ArrayList<Integer>> bucketArr = new ArrayList<>(bucketCount);
        ArrayList<Integer> resultArr = new ArrayList<>();
        for (int i = 0; i < bucketCount; i++) 
        {
            bucketArr.add(new ArrayList<Integer>());
        }
        for (int i = 0; i < array.size(); i++) 
        {
            bucketArr.get((array.get(i) - min) / bucketSize).add(array.get(i));
        }
        for (int i = 0; i < bucketCount; i++) 
        {
            if (bucketSize == 1) 
            { // 如果带排序数组中有重复数字时  感谢 @见风任然是风 朋友指出错误
                for (int j = 0; j < bucketArr.get(i).size(); j++)
                    resultArr.add(bucketArr.get(i).get(j));
            } else {
                if (bucketCount == 1)
                    bucketSize--;
                ArrayList<Integer> temp = BucketSort(bucketArr.get(i), bucketSize);
                for (int j = 0; j < temp.size(); j++)
                    resultArr.add(temp.get(j));
            }
        }
        return resultArr;
    }
```
## 基数排序（Radix Sort）
基数排序也是非比较的排序算法，对每一位进行排序，从最低位开始排序，复杂度为O(kn),为数组长度，k为数组中的数的最大的位数；

基数排序是按照低位先排序，然后收集；再按照高位排序，然后再收集；依次类推，直到最高位。有时候有些属性是有优先级顺序的，先按低优先级排序，再按高优先级排序。最后的次序就是高优先级高的在前，高优先级相同的低优先级高的在前。基数排序基于分别排序，分别收集，所以是稳定的。
### 算法描述
+ 取得数组中的最大数，并取得位数；
+ arr为原始数组，从最低位开始取每个位组成radix数组；
+ 对radix进行计数排序（利用计数排序适用于小范围数的特点）；
### 动画演示
![](https://s2.ax1x.com/2019/03/18/AnKf1K.gif)
### 代码实现
```
/**
     * 基数排序
     * @param array
     * @return
     */
    public static int[] RadixSort(int[] array) 
    {
        if (array == null || array.length < 2)
            return array;
        // 1.先算出最大数的位数；
        int max = array[0];
        for (int i = 1; i < array.length; i++) 
        {
            max = Math.max(max, array[i]);
        }
        int maxDigit = 0;
        while (max != 0) 
        {
            max /= 10;
            maxDigit++;
        }
        int mod = 10, div = 1;
        ArrayList<ArrayList<Integer>> bucketList = new ArrayList<ArrayList<Integer>>();
        for (int i = 0; i < 10; i++)
            bucketList.add(new ArrayList<Integer>());
        for (int i = 0; i < maxDigit; i++, mod *= 10, div *= 10) 
        {
            for (int j = 0; j < array.length; j++) 
            {
                int num = (array[j] % mod) / div;
                bucketList.get(num).add(array[j]);
            }
            int index = 0;
            for (int j = 0; j < bucketList.size(); j++) 
            {
                for (int k = 0; k < bucketList.get(j).size(); k++)
                    array[index++] = bucketList.get(j).get(k);
                bucketList.get(j).clear();
            }
        }
        return array;
    }
```
**基数排序 vs 计数排序 vs 桶排序**
这三种排序算法都利用了桶的概念，但对桶的使用方法上有明显差异：

+ 基数排序：根据键值的每位数字来分配桶
+ 计数排序：每个桶只存储单一键值
+ 桶排序：每个桶存储一定范围的数值

参考：
https://www.cnblogs.com/guoyaohua/p/8600214.html
