---
title: face_recognition试用
date: 2019-08-28 15:50:36
tags:
 - python
 - opencv
 - dlib
 - face_recognition
---


# 安装

```
pip install face_recognition
```

# 识别人脸


```python
# -*- coding: utf-8 -*-
# 

# 检测人脸
import face_recognition
import cv2

# 读取图片并识别人脸
img = face_recognition.load_image_file("faces/1.jpg")
face_locations = face_recognition.face_locations(img)
print(face_locations)

# 遍历每个人脸，并标注
faceNum = len(face_locations)
for i in range(0, faceNum):
    top =  face_locations[i][0]
    right =  face_locations[i][1]
    bottom = face_locations[i][2]
    left = face_locations[i][3]

    start = (left, top)
    end = (right, bottom)

    color = (55,255,155)
    thickness = 3
    cv2.rectangle(img, start, end, color, thickness)

# 显示识别结果
cv2.namedWindow("img2")
cv2.imshow("img2", img)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

    [(67, 296, 196, 167)]
    

# 人脸比对


```python
import face_recognition
jobs_image = face_recognition.load_image_file("faces/1.jpg");
obama_image = face_recognition.load_image_file("faces/3.jpg");
unknown_image = face_recognition.load_image_file("faces/2.jpg");

jobs_encoding = face_recognition.face_encodings(jobs_image)[0]
obama_encoding = face_recognition.face_encodings(obama_image)[0]
unknown_encoding = face_recognition.face_encodings(unknown_image)[0]

results = face_recognition.compare_faces([jobs_encoding, obama_encoding], unknown_encoding )
labels = ['obam', 'mibb']

print('results:'+str(results))

for i in range(0, len(results)):
    if results[i] == True:
        print('The person is:'+labels[i])
```

    results:[True, False]
    The person is:obam
    

识别出未知的照片是  obam

# 摄像头实时识别


```python
# -*- coding: utf-8 -*-
import face_recognition
import cv2

video_capture = cv2.VideoCapture('rtsp://admin:thrn.com@192.168.2.214:554/h264/ch33/main/av_stream')

obama_img = face_recognition.load_image_file("faces/3.jpg")
obama_face_encoding = face_recognition.face_encodings(obama_img)[0]

face_locations = []
face_encodings = []
face_names = []
process_this_frame = True

while True:
    ret, frame = video_capture.read()

    small_frame = cv2.resize(frame, (0, 0), fx=0.25, fy=0.25)

    if process_this_frame:
        face_locations = face_recognition.face_locations(small_frame)
        face_encodings = face_recognition.face_encodings(small_frame, face_locations)

        face_names = []
        for face_encoding in face_encodings:
            match = face_recognition.compare_faces([obama_face_encoding], face_encoding)

            if match[0]:
                name = "mibb"
            else:
                name = "unknown"

            face_names.append(name)

    process_this_frame = not process_this_frame

    for (top, right, bottom, left), name in zip(face_locations, face_names):
        top *= 4
        right *= 4
        bottom *= 4
        left *= 4

        cv2.rectangle(frame, (left, top), (right, bottom), (0, 0, 255),  2)

        cv2.rectangle(frame, (left, bottom - 35), (right, bottom), (0, 0, 255), 2)
        font = cv2.FONT_HERSHEY_DUPLEX
        cv2.putText(frame, name, (left+6, bottom-6), font, 1.0, (255, 255, 255), 1)

    cv2.imshow('Video', frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

video_capture.release()
cv2.destroyAllWindows()
```

