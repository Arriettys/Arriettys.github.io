---
title: OpenCV-画几何图
date: 2019-11-04 21:30:06
tags:
	- Python
	-OpenCV
categories: 图像处理
---

接下来我们要接触的函数：functions : [**cv.line()**](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html#ggaf076ef45de481ac96e0ab3dc2c29a777a85fdabe5335c9e6656563dfd7c94fb4f), [**cv.circle()**](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html#gaf10604b069374903dbd0f0488cb43670) , [**cv.rectangle()**](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html#ga07d2f74cadcf8e305e810ce8eed13bc9), [**cv.ellipse()**](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html#ga28b2267d35786f5f890ca167236cbc69), [**cv.putText()**](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html#ga5126f47f883d730f633d74f07456c576) 等.

这些函数都有一下相同的参数：

<!--more-->

+ img：你想画出的形状
+ color：色彩维度，比如BGR，以元组形式输入，如（255,0,0）
+ thickness：线或环得厚度，如果输入-1则是封闭图形，如圆。默认thickness为1
+ lineType：线性，注意不是指线型是实线、虚线还是点画线，这个参数实际用途是改变线的产生算法，默认是8通线

### 线

需要起点和终点

```Python
import numpy as np
import cv2 as cv
# Create a black image
img = np.zeros((512,512,3), np.uint8)
# Draw a diagonal blue line with thickness of 5 px
cv.line(img,(0,0),(511,511),(255,0,0),5)
```

### 长方形

需要四角

```Python
cv.rectangle(img,(384,0),(510,128),(0,255,0),3)
```

### 圆

需要圆心和半径，还可以填充颜色

```Python
cv.circle(img,(447,63), 63, (0,0,255), -1)
```

### 椭圆

需要一下参数

+ 中心位置（x, y）
+ 轴长（长轴，短轴）
+ 逆时针方向椭圆旋转角
+ 起始角度、终止角度、图色、线宽等

```
cv.ellipse(img,(256,256),(100,50),0,0,180,255,-1)
```

### 多边形

需要各顶点坐标，我们可以把它们放入rowsx1x2维度的矩阵中，设置类型为int32

```Python
pts = np.array([[10,5],[20,30],[70,20],[50,10]], np.int32)
pts = pts.reshape((-1,1,2))
cv.polylines(img,[pts],True,(0,255,255))
```

### 文字

需要指定一下事项：

+ 文本内容
+ 放置坐标
+ 字体
+ 字号
+ 字色，线宽，线型（lineType）

```python
font = cv.FONT_HERSHEY_SIMPLEX
cv.putText(img,'OpenCV',(10,500), font, 4,(255,255,255),2,cv.LINE_AA)
```



完整代码

```Python
import numpy as np
import cv2 as cv

import numpy as np
import cv2 as cv
# Create a black image
img = np.zeros((512,512,3), np.uint8)
# Draw a diagonal blue line with thickness of 5 px
cv.line(img,(0,0),(511,511),(255,0,0),5)

cv.rectangle(img,(384,0),(510,128),(0,255,0),3)

cv.circle(img,(447,63), 63, (0,0,255), -1)

cv.ellipse(img,(256,256),(100,50),0,360,0,100,-1)

# cv.ellipse(img,(256,256),(100,50),0,0,360,255,-1)

pts = np.array([[10,5],[20,30],[70,20],[50,10]], np.int32)
pts = pts.reshape((-1,1,2))
cv.polylines(img,[pts],True,(0,255,255))

font = cv.FONT_HERSHEY_SIMPLEX
cv.putText(img,'OpenCV',(10,500), font, 4,(255,255,255),2,cv.LINE_AA)

cv.imshow('frame', img)
cv.waitKey(5000)
```

![](https://s2.ax1x.com/2019/11/04/KzY47F.png)

