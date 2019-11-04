---
title: OpenCV-开始使用视频
date: 2019-11-04 19:46:13
tags:
	- Python
	- OpenCV
categories: 图像处理
---

## 获取视频

### cv.VideoCapture(path/device)

+ device：如果想从设备中获取，则参数输入设备下标，如0（or1），更多详细信息可以看[OpenCV官网](https://docs.opencv.org/master/dd/d43/tutorial_py_video_display.html)
+ path：从文件获取，则输入文件路径
+ return：VideoCapture对象

<!--more-->

```Python
import numpy as np
import cv2 as cv
cap = cv.VideoCapture('vtest.avi')
while cap.isOpened():
    ret, frame = cap.read()
    # if frame is read correctly ret is True
    if not ret:
        print("Can't receive frame (stream end?). Exiting ...")
        break
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cv.imshow('frame', gray)
    # 这里我们使用键盘事件制造时间间隔是为了不让帧切换太快达到正常播放速度
    if cv.waitKey(10) == ord('q'):
        break
cap.release()
cv.destroyAllWindows()
```

#### VideoCapture.read()

一帧帧读取，返回每帧图像frame和读取是否正确ret

#### VideoCapture.get(propId)

propId是数字0-18，分别代表video不同属性，详细参考[cv::VideoCapture::get()](https://docs.opencv.org/master/d8/dfe/classcv_1_1VideoCapture.html#aa6480e6972ef4c00d74814ec841a2939)，这里列举几个

cap.get([cv.CAP_PROP_FRAME_WIDTH](https://docs.opencv.org/master/d4/d15/group__videoio__flags__base.html#ggaeb8dd9c89c10a5c63c139bf7c4f5704dab26d2ba37086662261148e9fe93eecad))：读取video宽

cap.get([cv.CAP_PROP_FRAME_HEIGHT](https://docs.opencv.org/master/d4/d15/group__videoio__flags__base.html#ggaeb8dd9c89c10a5c63c139bf7c4f5704dad8b57083fd9bd58e0f94e68a54b42b7e))：读取video高

cap.set([cv.CAP_PROP_FRAME_WIDTH](https://docs.opencv.org/master/d4/d15/group__videoio__flags__base.html#ggaeb8dd9c89c10a5c63c139bf7c4f5704dab26d2ba37086662261148e9fe93eecad),320)和cap.set([cv.CAP_PROP_FRAME_HEIGHT](https://docs.opencv.org/master/d4/d15/group__videoio__flags__base.html#ggaeb8dd9c89c10a5c63c139bf7c4f5704dad8b57083fd9bd58e0f94e68a54b42b7e),240)则是设置



## 保存视频

### [cv.VideoWriter(path, fourCC, ...)](https://docs.opencv.org/master/dd/d9e/classcv_1_1VideoWriter.html)

+ path：保存路径，注意.后缀要和fourcc一致
+ fourcc：是一种独立标示视频数据流格式的四字符代码，视频播放软件通过查询 FourCC 代码并且寻找与 FourCC 代码相关联的视频解码器来播放特定的视频流
  + fourCC代码的传递：cv.VideoWriter_fource('M','J','P','G')或cv.VideoWriter_fourcc(*'MJPG')` 对应MJPG.
  + 更多fourCC细节参考[fourcc.org](http://www.fourcc.org/codecs.php)

```Python
import numpy as np
import cv2 as cv
cap = cv.VideoCapture('vtest.avi')
# Define the codec and create VideoWriter object
fourcc = cv.VideoWriter_fourcc(*'XVID')
out = cv.VideoWriter('output.avi', fourcc, 20.0, (640,  480))
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        print("Can't receive frame (stream end?). Exiting ...")
        break
    frame = cv.flip(frame, 0)
    # write the flipped frame
    out.write(frame)
    cv.imshow('frame', frame)
    if cv.waitKey(1) == ord('q'):
        break
# Release everything if job is finished
cap.release()
out.release()
cv.destroyAllWindows()
```

