---
title: OpenCV-开始使用图片
date: 2019-11-03 22:17:56
tags: 
	- Python
	- OpenCV
categories: 图像处理
---

## Using OpenCV

![](https://s2.ax1x.com/2019/11/03/Kj0gw8.png)

<!--more-->

### 读取图片

#### cv.imread(imgPath, flag)

imgPath：图片路径，字符串

flag：图片读取方式

- cv.IMREAD_COLOR：加载一张彩色图片，任何透明图像会被忽略，这是默认flag
- cv.IMREAD_GRAYSCALE：使用灰度模式加载图片
- cv.IMREAD_UNCHANGED：加载图像包括alpha通道



```python
import numpy as np
import cv2 as cv

img = cv.imread('img.jpg', cv.IMREAD_COLOR)
```

### 显示图片

#### cv.imshow(imgName, img)

imgName：你自己给张图的命名，字符串

img：cv读取到的图片，如上文的img



```python
cv.imshow('image', img)
k = cv.waitKey(0) # k 为按键值
cv.destroyAllWindow()
```

实际使用时，第一句只会弹出显示图片的窗口，但没有图片

![](https://s2.ax1x.com/2019/11/03/Kjmhut.png)

输入第二句的时候才会显示图片

![](https://s2.ax1x.com/2019/11/03/KjmOvn.png)

#### cv.waitKey(milliseconds)

绑定键盘点击事件，在指定时间（毫秒）内可中断程序，如果你是使用ipython的shell模式实验，会发现显示图片后你是不能在控制台输入的，因为主线程在显示图片这时候就需要你点击键盘退回控制台

![](https://s2.ax1x.com/2019/11/03/KjngaT.png)



若未在指定时间内点击程序会自动中断；若milliseconds设置为0则是无限期等待。

#### cv.destroyAllWindows()

中断后，虽然退回了控制台，但图片依然存在，使用该函数可以关闭所有窗口；若想关闭指定窗口可使用cv.destroyWindow(imgName).

#### cv.namedWindow(imgName, flag)

设置窗口大小

flag：

- cv.WINDOW_AUTOSIZE：自动适应图片大小
- cv.WINDOW_NORMAL：自设置窗口大小，图片尺寸过大时增加滑动条

```python
cv.namedWindow('image', cv.WINDOW_NORMAL)
cv.imshow('image',img)
k = cv.waitKey(0) # k 为按键值
cv.destroyAllWindows()
```



### 写入图片

#### cv.imwrite(imgPath, img)

imgPath：保存路径

img：图片对象



## Using Matplotlib

Python的Matplotlib库有各种各样的绘图方法，并且它能提供许多可选项

```Python
import numpy as np
import cv2 as cv
from matplotlib import pyplot as plt
img = cv.imread('messi5.jpg',0)
plt.imshow(img, cmap = 'gray', interpolation = 'bicubic')
plt.xticks([]), plt.yticks([])  # to hide tick values on X and Y axis
plt.show()
```



![](https://s2.ax1x.com/2019/11/03/KjdOBT.png)



ps.OpenCV读取彩色图片时使用的是BGR模式，而matplotlib则是RGB模式，所以一些OpenCV读取的彩色图片matplotlib显示会有问题