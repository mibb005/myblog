---
title: 面部标志探测器
date: 2019-08-27 16:40:51
tags:
 - Dlib
 - opencv
 - Python
---


# Dlib的5点面部标志探测器


当68点探测器定位沿着眼睛，眉毛，鼻子，嘴巴和下颚线的区域时，5点面部标志探测器将此信息减少为：

* 左眼2分
* 右眼2分
* 1分为鼻子

5点探测器比原始版本快8-10％，模型尺寸：分别为9.2MB和99.7MB（小于10倍）。


通常，在选择面部检测模型时，您会发现以下指南是一个很好的起点：

* Haar级联：快速，但不太准确。调整参数可能很痛苦。
* HOG +线性SVM：通常（显着）比Haar级联更准确，误报率更低。通常在测试时调整的参数较少。与Haar级联相比可能会很慢。
* 基于深度学习的探测器：正确训练后，比Haar级联和HOG +线性SVM更加准确和稳健。根据模型的深度和复杂程度，可能会非常慢。可以通过在GPU上进行推理来加速（你可以在这篇文章中看到一个OpenCV深度学习面部探测器）。

# 使用dlib，OpenCV和Python实现面部标记

## import 


```python
# import the necessary packages
from imutils.video import VideoStream
from imutils import face_utils
import argparse
import imutils
import time
import dlib
import cv2
```


```python
# pip install --upgrade imutils
```

## 初始化视频流 


```python
# initialize dlib's face detector (HOG-based) and then create the
# facial landmark predictor
print("[INFO] loading facial landmark predictor...")
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor('cnn/shape_predictor_68_face_landmarks.dat')
 
# initialize the video stream and sleep for a bit, allowing the
# camera sensor to warm up
print("[INFO] camera sensor warming up...")
vs = VideoStream(src='rtsp://admin:thrn.com@192.168.2.214:554/h264/ch33/main/av_stream').start()
# vs = VideoStream(usePiCamera=True).start() # Raspberry Pi
time.sleep(2.0)
```

    [INFO] loading facial landmark predictor...
    [INFO] camera sensor warming up...
    

## 启动跟踪


```python
# loop over the frames from the video stream
while True:
	# grab the frame from the threaded video stream, resize it to
	# have a maximum width of 400 pixels, and convert it to
	# grayscale
	frame = vs.read()
	frame = imutils.resize(frame, width=800)
	gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

	# detect faces in the grayscale frame
	rects = detector(gray, 0)
	# check to see if a face was detected, and if so, draw the total
	# number of faces on the frame
	if len(rects) > 0:
		text = "{} face(s) found".format(len(rects))      
		cv2.putText(frame, text, (10, 20), cv2.FONT_HERSHEY_SIMPLEX,
			0.5, (0, 0, 255), 2)
	# loop over the face detections
	for rect in rects:
		# compute the bounding box of the face and draw it on the
		# frame
		(bX, bY, bW, bH) = face_utils.rect_to_bb(rect)
		cv2.rectangle(frame, (bX, bY), (bX + bW, bY + bH),
			(0, 255, 0), 1)
 
		# determine the facial landmarks for the face region, then
		# convert the facial landmark (x, y)-coordinates to a NumPy
		# array
		shape = predictor(gray, rect)
		shape = face_utils.shape_to_np(shape)
 
		# loop over the (x, y)-coordinates for the facial landmarks
		# and draw each of them
		for (i, (x, y)) in enumerate(shape):
			cv2.circle(frame, (x, y), 1, (0, 0, 255), -1)
			cv2.putText(frame, str(i + 1), (x - 10, y - 10),
				cv2.FONT_HERSHEY_SIMPLEX, 0.35, (0, 0, 255), 1)
	# show the frame
	cv2.imshow("Frame", frame)
	key = cv2.waitKey(1) & 0xFF
 
	# if the `q` key was pressed, break from the loop
	if key == ord("q"):
		break
 
# do a bit of cleanup
cv2.destroyAllWindows()
vs.stop()
```

# dlib的5点或68点面部标志检测器更快吗？

在我自己的测试中，我发现dlib的5点面部标志探测器比原来的68点面部标志探测器快8-10％。

加速8-10％是显着的; 但是，更重要的是模型的大小。

最初的68点面部地标接近100MB，重达99.7MB。

5点面部标志探测器低于10MB，仅为9.2MB - 超过10倍的小型号！

当您构建自己的应用程序使用面部标记时，您现在拥有一个小得多的模型文件，可以与您的应用程序的其余部分一起分发。

较小的型号尺寸也无需嘲笑 - 只需考虑移动应用用户的下载时间/资源减少！

# 5点面部标志探测器的局限性

5点面部标志检测器的主要用途是面部对齐：

对于面部对齐，可以将5点面部标志检测器视为68点检测器的直接替代品 - 相同的通用算​​法适用：

* 计算5点面部标志
* 根据每只眼睛的两个界标分别计算每只眼睛的中心
* 通过利用眼睛之间的中点来计算眼睛质心之间的角度
* 通过应用仿射变换获得面部的规范对齐
* 虽然68点面部标志探测器可以给我们稍微更好的近似眼睛中心，但实际上你会发现5点面部标志探测器也能正常工作。

尽管如此，虽然5点面部标志探测器肯定更小（分别为9.2MB和99.7MB），但并不能在所有情况下使用。

