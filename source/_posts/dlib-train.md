---
title: dlib深度学习模型训练
date: 2019-08-28 14:37:03
tags:
 - dlib
 - python
 - opencv
---

# 下载

https://github.com/davisking/dlib

# win10 编译imglab

* 进入`tools\imglab`目录
* 新建文件build，进入build
* 右键cmd命令，输入：`cmake ..`
* cmd命令，输入：`cmake --build . --config Release`
* 将`Release`目录加入到系统环境变量

```
mkdir build
cd build
cmake ..
cmake --build . --config Release
```

编译完成

# imglab图片标注

* 新建文件夹：`images `
* 在`images`下放你想标注的图像，就是训练集
* 输入：`imglab -c mydataset.xml images`，生成`mydataset.xml`文件
* cmd输入：`imglab mydataset.xml`
* 出现`imglab`标注软件了，可以自己标注了，这是一个漫长的过程
* 打标注，打标签，保存，退出软件

xml其实就是保存标注点的坐标信息了

# 训练模型


```python
# %load train_model.py
import os
import sys
import glob
import dlib
import cv2

# options用于设置训练的参数和模式
options = dlib.simple_object_detector_training_options()
# Since faces are left/right symmetric we can tell the trainer to train a
# symmetric detector.  This helps it get the most value out of the training
# data.
options.add_left_right_image_flips = True
# 支持向量机的C参数，通常默认取为5.自己适当更改参数以达到最好的效果
options.C = 5
# 线程数，你电脑有4核的话就填4
options.num_threads = 4
options.be_verbose = True

# 获取路径
current_path = os.getcwd()
train_folder = current_path + '/train/cats_train/'
test_folder = current_path + '/train/cats_test/'
train_xml_path = train_folder + 'mydataset.xml'
test_xml_path = test_folder + 'mydataset.xml'

print("training file path:" + train_xml_path)
# print(train_xml_path)
print("testing file path:" + test_xml_path)
# print(test_xml_path)

# 开始训练
print("start training:")
dlib.train_simple_object_detector(train_xml_path, 'detector.svm', options)

print("")  # Print blank line to create gap from previous output
print("Training accuracy: {}".format(
    dlib.test_simple_object_detector(train_xml_path, "detector.svm")))

print("Testing accuracy: {}".format(
    dlib.test_simple_object_detector(test_xml_path, "detector.svm")))
```

    training file path:G:\py\dlib/train/cats_train/mydataset.xml
    testing file path:G:\py\dlib/train/cats_test/mydataset.xml
    start training:
    
    Training accuracy: precision: 1, recall: 1, average precision: 1
    Testing accuracy: precision: 1, recall: 1, average precision: 1
    

# 测试模型


```python
# %load test_model.py

import os
import sys
import dlib
import cv2
import glob
print('start')
detector = dlib.simple_object_detector("detector.svm")

current_path = os.getcwd()
test_folder = current_path + '/train/cats_train/images/'
print('load images')
for f in glob.glob(test_folder+'*.jpg'):
    print("Processing file: {}".format(f))
    img = cv2.imread(f, cv2.IMREAD_COLOR)
    b, g, r = cv2.split(img)
    img2 = cv2.merge([r, g, b])
    dets = detector(img2)
    print("Number of faces detected: {}".format(len(dets)))
    for index, face in enumerate(dets):
        print('face {}; left {}; top {}; right {}; bottom {}'.format(index, face.left(), face.top(), face.right(), face.bottom()))

        left = face.left()
        top = face.top()
        right = face.right()
        bottom = face.bottom()
        cv2.rectangle(img, (left, top), (right, bottom), (0, 255, 0), 3)
        cv2.namedWindow(f, cv2.WINDOW_AUTOSIZE)
        cv2.imshow(f, img)

k = cv2.waitKey(0)
cv2.destroyAllWindows()
```

    start
    load images
    Processing file: G:\py\dlib/train/cats_train/images\filename1.jpg
    Number of faces detected: 1
    face 0; left 681; top 194; right 910; bottom 480
    Processing file: G:\py\dlib/train/cats_train/images\filename10.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename11.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename12.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename13.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename14.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename15.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename16.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename17.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename18.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename19.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename2.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename20.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename21.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename22.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename23.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename24.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename25.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename26.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename27.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename28.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename29.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename3.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename30.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename31.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename32.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename33.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename4.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename5.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename6.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename7.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename8.jpg
    Number of faces detected: 0
    Processing file: G:\py\dlib/train/cats_train/images\filename9.jpg
    Number of faces detected: 0
    
