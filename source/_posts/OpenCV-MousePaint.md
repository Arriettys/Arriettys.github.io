---
title: OpenCV-鼠标作画和滑块
date: 2019-11-05 22:11:48
tags:
	-Python
	-OpenCV
categories: 图像处理
---

## 鼠标事件

鼠标在窗口上作画的基础是鼠标事件的回调函数，鼠标事件能为我们提供坐标地址，之后我们就能做许多事情，下列代码能列出所有可用的事件

```Python
import cv2 as cv
events = [i for i in dir(cv) if 'EVENT' in i]
print( events )
```

<!--more-->

接下来，我们用鼠标双击事件画圆

```Python
import numpy as np
import cv2 as cv
# mouse callback function
def draw_circle(event,x,y,flags,param):
    if event == cv.EVENT_LBUTTONDBLCLK:
        cv.circle(img,(x,y),100,(255,0,0),-1)
# Create a black image, a window and bind the function to window
img = np.zeros((512,512,3), np.uint8)
cv.namedWindow('image')
cv.setMouseCallback('image',draw_circle)
while(1):
    cv.imshow('image',img)
    if cv.waitKey(20) & 0xFF == 27: # Esc
        break
cv.destroyAllWindows()
```



更高级点的例子，根据你的决定选择画圆还是画矩形。这里用到了三个鼠标事件，鼠标移动计算坐标，左键按下开始画图，松开停止

```Python
import numpy as np
import cv2 as cv
drawing = False # true if mouse is pressed
mode = True # if True, draw rectangle. Press 'm' to toggle to curve
ix,iy = -1,-1
# mouse callback function
def draw_circle(event,x,y,flags,param):
    global ix,iy,drawing,mode
    if event == cv.EVENT_LBUTTONDOWN:
        drawing = True
        ix,iy = x,y
    elif event == cv.EVENT_MOUSEMOVE:
        if drawing == True:
            if mode == True:
                cv.rectangle(img,(ix,iy),(x,y),(0,255,0),-1)
            else:
                cv.circle(img,(x,y),5,(0,0,255),-1)
    elif event == cv.EVENT_LBUTTONUP:
        drawing = False
        if mode == True:
            cv.rectangle(img,(ix,iy),(x,y),(0,255,0),-1)
        else:
            cv.circle(img,(x,y),5,(0,0,255),-1)
```

随后将回调函数和OpenCV窗口绑定在一起

```Python
img = np.zeros((512,512,3), np.uint8)
cv.namedWindow('image')
cv.setMouseCallback('image',draw_circle)
while(1):
    cv.imshow('image',img)
    k = cv.waitKey(1) & 0xFF
    if k == ord('m'):
        mode = not mode
    elif k == 27:
        break
cv.destroyAllWindows()
```

![](https://s2.ax1x.com/2019/11/05/M96QOO.png)



## 滑块

在这里,我们将创建一个简单的应用程序,它显示了你指定的颜色，在OpenCV窗口中展示色彩和三基色B,G,R的滑块，通过滑动BGR的滑块引起色彩的变化，这里还用滑块做了一个开关，通过绑定回调函数起到控制的作用

用到的函数：

[cv.createTrackbar](https://docs.opencv.org/master/d7/dfc/group__highgui.html#gaf78d2155d30b728fc413803745b67a9b)(trackbarName, imgName, defaultValue, maxValue, callbackFunction)

[cv.getTrackbarPos](https://docs.opencv.org/master/d7/dfc/group__highgui.html#ga122632e9e91b9ec06943472c55d9cda8)(trackbarName, imgName)

```Python
import numpy as np
import cv2 as cv
def nothing(x):
    pass
# Create a black image, a window
img = np.zeros((300,512,3), np.uint8)
cv.namedWindow('image')
# create trackbars for color change
cv.createTrackbar('R','image',0,255,nothing)
cv.createTrackbar('G','image',0,255,nothing)
cv.createTrackbar('B','image',0,255,nothing)
# create switch for ON/OFF functionality
switch = '0 : OFF \n1 : ON'
cv.createTrackbar(switch, 'image',0,1,nothing)
while(1):
    cv.imshow('image',img)
    k = cv.waitKey(1) & 0xFF
    if k == 27:
        break
    # get current positions of four trackbars
    r = cv.getTrackbarPos('R','image')
    g = cv.getTrackbarPos('G','image')
    b = cv.getTrackbarPos('B','image')
    s = cv.getTrackbarPos(switch,'image')
    if s == 0:
        img[:] = 0
    else:
        img[:] = [b,g,r]
cv.destroyAllWindows()
```

![](https://s2.ax1x.com/2019/11/05/M9Rkg1.png)

