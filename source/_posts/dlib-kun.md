---
title: 困倦探测
date: 2019-08-27 17:36:26
tags:
 - dlib
 - opencv
 - python
---


# 困倦探测器算法

我们的困倦检测算法的一般流程相当简单。

首先，我们将设置一个监视面部流的摄像头：

如果找到了面部，我们应用面部标志检测并提取眼部区域：

现在我们有眼睛区域，我们可以计算眼睛纵横比，以确定眼睛是否闭合：


```python
# import the necessary packages
from scipy.spatial import distance as dist
from imutils.video import VideoStream
from imutils import face_utils
import numpy as np
import imutils
import time
import dlib
import cv2
```

`eye_aspect_ratio` 函数，该函数用于计算垂直眼睛地标与水平眼睛地标之间距离的比率：


```python
def eye_aspect_ratio(eye):
	# compute the euclidean distances between the two sets of
	# vertical eye landmarks (x, y)-coordinates
	A = dist.euclidean(eye[1], eye[5])
	B = dist.euclidean(eye[2], eye[4])
 
	# compute the euclidean distance between the horizontal
	# eye landmark (x, y)-coordinates
	C = dist.euclidean(eye[0], eye[3])
 
	# compute the eye aspect ratio
	ear = (A + B) / (2.0 * C)
 
	# return the eye aspect ratio
	return ear
```

当眼睛打开时，眼睛纵横比的返回值将近似恒定。然后，该值将在眨眼期间快速减小到零。

如果眼睛闭合，眼睛纵横比将再次保持近似恒定，但将  远小于眼睛打开时的比率。

在我们的困倦探测器情况下，我们将监测眼睛纵横比，看看该值是否  下降但  不再增加，从而暗示该人已闭上眼睛。

定义参数  EYE_AR_THRESH 默认0.3


```python
# define two constants, one for the eye aspect ratio to indicate
# blink and then a second constant for the number of consecutive
# frames the eye must be below the threshold for to set off the
# alarm
EYE_AR_THRESH = 0.3
EYE_AR_CONSEC_FRAMES = 48
 
# initialize the frame counter as well as a boolean used to
# indicate if the alarm is going off
COUNTER = 0
ALARM_ON = False
```

dlib库附带了一个基于方向梯度的直方图面部检测器以及面部标志预测器  - 我们在以下代码块中实例化这两个：


```python
# initialize dlib's face detector (HOG-based) and then create
# the facial landmark predictor
print("[INFO] loading facial landmark predictor...")
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor('cnn/shape_predictor_68_face_landmarks.dat')
```

    [INFO] loading facial landmark predictor...
    

dlib生成的面部标志是一个可索引的列表，我在[这里](https://www.pyimagesearch.com/2017/04/10/detect-eyes-nose-lips-jaw-dlib-opencv-python/)描述：

![你想输入的替代文字](3.jpg)

因此，要从一组面部标志中提取眼睛区域，我们只需要知道正确的阵列切片索引：


```python
# grab the indexes of the facial landmarks for the left and
# right eye, respectively
(lStart, lEnd) = face_utils.FACIAL_LANDMARKS_IDXS["left_eye"]
(rStart, rEnd) = face_utils.FACIAL_LANDMARKS_IDXS["right_eye"]
```


```python
# start the video stream thread
print("[INFO] starting video stream thread...")
vs = VideoStream(src='rtsp://admin:thrn.com@192.168.2.214:554/h264/ch33/main/av_stream').start()
time.sleep(1.0)
 
# loop over frames from the video stream
while True:
	# grab the frame from the threaded video file stream, resize
	# it, and convert it to grayscale
	# channels)
	frame = vs.read()
	frame = imutils.resize(frame, width=450)
	gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
 
	# detect faces in the grayscale frame
	rects = detector(gray, 0)
	# loop over the face detections
	for rect in rects:
		# determine the facial landmarks for the face region, then
		# convert the facial landmark (x, y)-coordinates to a NumPy
		# array
		shape = predictor(gray, rect)
		shape = face_utils.shape_to_np(shape)
 
		# extract the left and right eye coordinates, then use the
		# coordinates to compute the eye aspect ratio for both eyes
		leftEye = shape[lStart:lEnd]
		rightEye = shape[rStart:rEnd]
		leftEAR = eye_aspect_ratio(leftEye)
		rightEAR = eye_aspect_ratio(rightEye)
 
		# average the eye aspect ratio together for both eyes
		ear = (leftEAR + rightEAR) / 2.0
		# compute the convex hull for the left and right eye, then
		# visualize each of the eyes
		leftEyeHull = cv2.convexHull(leftEye)
		rightEyeHull = cv2.convexHull(rightEye)
		cv2.drawContours(frame, [leftEyeHull], -1, (0, 255, 0), 1)
		cv2.drawContours(frame, [rightEyeHull], -1, (0, 255, 0), 1)
		# check to see if the eye aspect ratio is below the blink
		# threshold, and if so, increment the blink frame counter
		if ear < EYE_AR_THRESH:
			COUNTER += 1
 
			# if the eyes were closed for a sufficient number of
			# then sound the alarm
			if COUNTER >= EYE_AR_CONSEC_FRAMES:
				# if the alarm is not on, turn it on
# 				if not ALARM_ON:
# 					ALARM_ON = True
 
# 					# check to see if an alarm file was supplied,
# 					# and if so, start a thread to have the alarm
# 					# sound played in the background
# 					if args["alarm"] != "":
# 						t = Thread(target=sound_alarm,
# 							args=(args["alarm"],))
# 						t.deamon = True
# 						t.start()
 
				# draw an alarm on the frame
				cv2.putText(frame, "DROWSINESS ALERT!", (10, 30),
					cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)
 
		# otherwise, the eye aspect ratio is not below the blink
		# threshold, so reset the counter and alarm
		else:
			COUNTER = 0
			ALARM_ON = False
		# draw the computed eye aspect ratio on the frame to help
		# with debugging and setting the correct eye aspect ratio
		# thresholds and frame counters
		cv2.putText(frame, "EAR: {:.2f}".format(ear), (300, 30),
			cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)
 
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

    [INFO] starting video stream thread...
    

