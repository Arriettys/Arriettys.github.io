---
title: OpenCV-BasicOperationOnImage
date: 2019-11-06 20:17:51
tags:
	-Python
	-OpenCV
categories: 图像处理
---

## 图片对象操作

几乎所有的操作在本节主要是有关Numpy而不是OpenCV，所以掌握好Numpy有利于写好OpenCV代码

<!--more-->

### 访问和操作图片像素

对于BGR图片来说，它的像素组成可以用一个三维矩阵表示，行列以及三基色值，如

![](https://s2.ax1x.com/2019/11/06/MiYIc4.png)

事实上numpy库最大优势是优化矩阵运算，所以进行简单的像素访问修改，反而会有些缓慢

对于单个像素的访问修改我们可以使用Numpy 矩阵方法：array.item()和array.itemset

![](https://s2.ax1x.com/2019/11/06/MiNCzF.png)

### 图片对象属性

这些属性包括行列、像素通道、图像数据类型(dtype调试时是非常重要的,因为大量的OpenCV-Python代码中的错误是由于无效的数据类型)、像素数量等，值得注意的是，正常图片shape属性会返回行列数、通道数，而灰度图片只有行列数

![QevsH0.png](https://s2.ax1x.com/2019/12/01/QevsH0.png)



### 图片ROI

ROI是region of images首字母的简写，翻译为图片的区域。有时处理图片的时候只对其中的一部分图片的区域进行处理，例如我们想在图片某个区域打马赛克，为了性能考虑我们可以只让程序对这一部分信息进行处理而将其他部分忽略，这时我们就要设置图片感性趣的区域。设置完感性趣的区域后其实是指针指到了ROI区域的左上角，好像我们截取了一张小图片一样，我们只对这张小图片进行处理就可以了，因其ROI指向的还是原图只在告诉它图片的起始位置和大小变了，所以在对ROI区操作会影响原图。

```python
import numpy as np
import cv2 as cv
img = cv.imread('image.jpg')
ball = img[280:340, 330:390]
cv.imshow('image1', img)
cv.imshow('image2', ball)
cv.waitKey(0)
```

[![QevoHx.md.png](https://s2.ax1x.com/2019/12/01/QevoHx.md.png)](https://imgse.com/i/QevoHx)

### 分裂和合并图像通道

有时你需要对图像的单通道进行操作，这种情况你可以从图片的矩阵中分割出单独通道矩阵进行操作，或者将几个单独的通道矩阵合并

```python 
b, g, r = cv.split(img) 
# 或者 b = img[:, :, 0]
# cv.split操作效率低，建议多使用Numpy array的操作
img = cv.merge((b, g, r))
```



### 制作图像边界

如果你想要给图像加上相框一样的边界，可以使用 **cv.copyMakeBorder()**，这个方法还可以用作卷积和零填充等方面上，它需要如下参数：

+ **src**：图片对象
+ **top, bottom, left, right**：相应方向上边界像素大小
+ borderType：边界类型，有一下几种
  + **[cv.BORDER_CONSTANT](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5aed2e4346047e265c8c5a6d0276dcd838)**：色彩的边界。色彩值应该为下一个参数
  + [**cv.BORDER_REFLECT**](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5a815c8a89b7cb206dcba14d11b7560f4b)：边界将镜面反射边界的元素，像这样：*fedcba|abcdefgh|hgfedcb*
  + **[cv.BORDER_REFLECT_101](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5ab3c5a6143d8120b95005fa7105a10bb4)** or **[cv.BORDER_DEFAULT](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5afe14c13a4ea8b8e3b3ef399013dbae01)**：同上，但有些不同，像：*gfedcb|abcdefgh|gfedcba*
  + [**cv.BORDER_REPLICATE**](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5aa1de4cff95e3377d6d0cbe7569bd4e9f)：最后的元素将被一直复制，像这样：*aaaaaa|abcdefgh|hhhhhhh*
  + [**cv.BORDER_WRAP**](https://docs.opencv.org/master/d2/de8/group__core__array.html#gga209f2f4869e304c82d07739337eae7c5a697c1b011884a7c2bdc0e5caf7955661)：
+ **value**：边界色彩

```Python
import cv2 as cv
import numpy as np
from matplotlib import pyplot as plt
BLUE = [255,0,0]
img1 = cv.imread('opencv-logo.png')
replicate = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REPLICATE)
reflect = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REFLECT)
reflect101 = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_REFLECT_101)
wrap = cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_WRAP)
constant= cv.copyMakeBorder(img1,10,10,10,10,cv.BORDER_CONSTANT,value=BLUE)
plt.subplot(231),plt.imshow(img1,'gray'),plt.title('ORIGINAL')
plt.subplot(232),plt.imshow(replicate,'gray'),plt.title('REPLICATE')
plt.subplot(233),plt.imshow(reflect,'gray'),plt.title('REFLECT')
plt.subplot(234),plt.imshow(reflect101,'gray'),plt.title('REFLECT_101')
plt.subplot(235),plt.imshow(wrap,'gray'),plt.title('WRAP')
plt.subplot(236),plt.imshow(constant,'gray'),plt.title('CONSTANT')
plt.show()
```

![](https://s2.ax1x.com/2019/11/06/Miclyq.png)

## 图像运算

### 图像加法

你可以使用OpenCV自带的方法 [cv.add()](https://docs.opencv.org/master/d2/de8/group__core__array.html#ga10ac1bfb180e2cfda1701d06c24fdbd6) ，也可以直接使用Numpy 的矩阵操作，但需要注意的是，OpenCV方法是一种饱和式操作（不超过255），Numpy则是一种模减运算

```Python
>>> x = np.uint8([250])
>>> y = np.uint8([10])
>>> print( cv.add(x,y) ) # 250+10 = 260 => 255
[[255]]
>>> print( x+y )          # 250+10 = 260 % 256 = 4 
[4]
```

ps.OpenCV方法能提供更好的结果，尽量使用使用它

### 图像混合

某种意义上来说也是一种加法，不同的是它通过给设置每张图片不同的比重，制造一种混合和透明的感觉

公式：


$$
g(x) = (1 - α)f_0 + αf_1(x)
$$

```python
img1 = cv.imread('img1.jpg')
img2 = cv.imread('img2.jpg')
dst = cv.addWeighted(img1,0.7,img2,0.3,0)
cv.imshow('dst',dst)
cv.waitKey(0)
cv.destroyAllWindows()
```

![](https://s2.ax1x.com/2019/11/07/MAhuxU.png)

主要方法：[cv.addWeighted()](https://docs.opencv.org/master/d2/de8/group__core__array.html#gafafb2513349db3bcff51f54ee5592a19)

ps.两张的尺寸要相同，否则矩阵运算会出错

### 逐位运算

这包括与、或、非和异或操作，在对图片做提取、定义和操作非矩形ROI等操作时经常用到。接下来让我们看看怎样改变图像的特定区域。

我想在例图上添加OpenCV logo，假如使用叠加，那么OpenCV的黑色背景会覆盖例图；假如使用混合则会造成透明效果，这是可以使用逐位运算解决

```Python
# 读取图片
img1 = cv.imread('messi5.jpg')
img2 = cv.imread('opencv-logo-white.png')
# 因为我想把logo放在右上角，所以我需要创建一个ROI
rows,cols,channels = img2.shape
roi = img1[0:rows, 0:cols]
# 将img2由BGR色域转为GRAY
img2gray = cv.cvtColor(img2,cv.COLOR_BGR2GRAY)
# 阀值分割 
ret, mask = cv.threshold(img2gray, 10, 255, cv.THRESH_BINARY)
mask_inv = cv.bitwise_not(mask)

img1_bg = cv.bitwise_and(roi, roi, mask = mask_inv)
img2_fg = cv.bitwise_and(img2, img2, mask = mask)
dst = cv.add(img1_bg, img2_fg)
img1[0:rows, 0:cols ] = dst
cv.imshow('res',img1)
cv.waitKey(0)
cv.destroyAllWindows()
```



上述代码中cv.cvtColor()将img2由BGR色域转为GRAY

![Qmw9eO.png](https://s2.ax1x.com/2019/12/01/Qmw9eO.png)

cv.threshold()设置阀值

![QmwUmT.png](https://s2.ax1x.com/2019/12/01/QmwUmT.png)

cv.bitwise_not()进行异位操作

![QmwkYd.png](https://s2.ax1x.com/2019/12/01/QmwkYd.png)

这样便可以在背景图与logo图中精准抠图，在相互叠加

[![QmGg3T.md.png](https://s2.ax1x.com/2019/12/01/QmGg3T.md.png)