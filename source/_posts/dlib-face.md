---
title: 人脸比对
date: 2019-08-28 11:18:48
tags:
 - dlib
 - Python
 - opencv
---

# 原理

使用dlib标定人脸，提取68个特征点，将人脸的信息提取成一个128维的向量空间。在这个向量空间上，同一个人脸的更接近，不同人脸的距离更远。度量采用欧式距离，欧氏距离计算不算复杂。

二维情况下： 

$$ \sqrt{(x1−x2)^2+(y1−y2)^2} $$

三维情况下： 

$$ \sqrt{(x1−x2)^2+(y1−y2)^2+(z1−z2)^2} $$

将其扩展到128维的情况下即可。 
通常使用的判别阈值是0.6，即如果两个人脸的向量空间的欧式距离超过了0.6，即认定不是同一个人；如果欧氏距离小于0.6，则认为是同一个人。这个距离也可以由自己定，只要效果能更好。

# 计算人脸128维向量


```python
# %load demo5.py
import sys
import dlib
import cv2
import os
import glob
import matplotlib.pyplot as plt

current_path = os.getcwd()  # 获取当前路径
# 模型路径
predictor_path = current_path + "\\cnn\\shape_predictor_68_face_landmarks.dat"
face_rec_model_path = current_path + "\\cnn\\dlib_face_recognition_resnet_model_v1.dat"
#测试图片路径
faces_folder_path = current_path + "\\faces\\"

# 读入模型
detector = dlib.get_frontal_face_detector()
shape_predictor = dlib.shape_predictor(predictor_path)
face_rec_model = dlib.face_recognition_model_v1(face_rec_model_path)

for img_path in glob.glob(os.path.join(faces_folder_path, "*.jpg")):
    print("Processing file: {}".format(img_path))
    # opencv 读取图片，并显示
    img = cv2.imread(img_path, cv2.IMREAD_COLOR)
    # opencv的bgr格式图片转换成rgb格式
    b, g, r = cv2.split(img)
    img2 = cv2.merge([r, g, b])

    dets = detector(img, 1)   # 人脸标定
    print("Number of faces detected: {}".format(len(dets)))

    for index, face in enumerate(dets):
        print('face {}; left {}; top {}; right {}; bottom {}'.format(index, face.left(), face.top(), face.right(), face.bottom()))

        shape = shape_predictor(img2, face)   # 提取68个特征点
        for i, pt in enumerate(shape.parts()):
            #print('Part {}: {}'.format(i, pt))
            pt_pos = (pt.x, pt.y)
            cv2.circle(img, pt_pos, 2, (255, 0, 0), 1)
            #print(type(pt))
        #print("Part 0: {}, Part 1: {} ...".format(shape.part(0), shape.part(1)))
#         cv2.namedWindow(img_path+str(index), cv2.WINDOW_AUTOSIZE)
#         cv2.imshow(img_path+str(index), img)
        plt.figure(img_path+str(index)) # 图像窗口名称
        plt.imshow(img)
        plt.axis('on') # 关掉坐标轴为 off
        plt.title('image') # 图像题目
        plt.show()

        face_descriptor = face_rec_model.compute_face_descriptor(img2, shape)   # 计算人脸的128维的向量
        print(face_descriptor)

k = cv2.waitKey(0)
cv2.destroyAllWindows()
```

    Processing file: G:\py\dlib\faces\1.jpg
    Number of faces detected: 1
    face 0; left 167; top 67; right 296; bottom 196
    


![png](output_3_1.png)


    -0.100903
    0.107215
    0.0127989
    -0.0740284
    0.0066496
    0.0225189
    -0.078312
    -0.0923171
    0.196753
    -0.108072
    0.236062
    0.101153
    -0.207568
    -0.139876
    0.0408826
    0.114068
    -0.199692
    -0.0939411
    -0.0996753
    -0.11982
    0.0413545
    -0.00487671
    0.0877109
    0.0492948
    -0.137577
    -0.353298
    -0.0608
    -0.171296
    0.00199143
    -0.105018
    -0.081396
    0.000772364
    -0.181964
    -0.107023
    0.0216598
    -0.0236119
    0.019205
    0.00282077
    0.201093
    0.0315438
    -0.125799
    0.0942641
    0.0268356
    0.230741
    0.276493
    0.0833094
    -0.00933838
    -0.0849845
    0.126553
    -0.229523
    0.0602546
    0.163942
    0.0720226
    0.0233181
    0.0970623
    -0.198381
    -0.0020905
    0.0887514
    -0.13795
    0.0311375
    0.00210361
    -0.0886309
    -0.0286811
    0.0405957
    0.191806
    0.0950056
    -0.124005
    -0.0708959
    0.129652
    -0.0221997
    0.022522
    0.0184984
    -0.194719
    -0.214093
    -0.233502
    0.0845566
    0.38484
    0.193726
    -0.198322
    0.0328681
    -0.202065
    0.0332264
    0.085176
    0.0261305
    -0.0700808
    -0.122556
    -0.0421356
    0.0571684
    0.0831238
    0.0534201
    -0.0515448
    0.228289
    -0.0252842
    0.0643284
    0.016167
    0.0649517
    -0.162603
    -0.00856679
    -0.17663
    -0.0593192
    0.0167162
    -0.0178645
    0.0543968
    0.145731
    -0.22838
    0.0544676
    0.00459545
    -0.0257714
    0.018232
    0.0707982
    -0.0418243
    -0.0374448
    0.0511305
    -0.254493
    0.23714
    0.253834
    0.0390552
    0.180478
    0.0882472
    0.00598699
    0.00265156
    -0.0281207
    -0.176554
    -0.0485108
    0.0544809
    0.0590729
    0.0764981
    0.00331541
    Processing file: G:\py\dlib\faces\2.jpg
    Number of faces detected: 1
    face 0; left 210; top 110; right 339; bottom 239
    


![png](output_3_3.png)


    -0.0656277
    0.103886
    0.0584333
    -0.00946767
    0.0313492
    0.00599562
    -0.0815832
    -0.0543527
    0.210806
    -0.0652889
    0.274125
    0.0799883
    -0.157585
    -0.127232
    0.0448985
    0.153828
    -0.150277
    -0.0207188
    -0.14031
    -0.0766388
    -6.17933e-06
    0.000160489
    0.0753391
    0.0159993
    -0.173483
    -0.378838
    -0.0494293
    -0.15175
    0.00172043
    -0.152177
    -0.0613851
    0.00298419
    -0.227171
    -0.0889237
    -0.0430099
    -0.0198571
    -0.00600264
    -0.0256885
    0.116474
    0.0501233
    -0.157516
    0.112152
    -0.0676546
    0.234205
    0.239259
    0.0355268
    0.0269477
    -0.0462278
    0.0857875
    -0.218189
    0.0237351
    0.0650885
    0.0457599
    -0.0163889
    0.0886727
    -0.14048
    -0.0469625
    0.132312
    -0.168475
    0.0611648
    -0.0031102
    -0.0756179
    -0.0682836
    -0.00150348
    0.179482
    0.137031
    -0.119424
    -0.0555014
    0.145337
    -0.00377072
    0.0697277
    -0.00566657
    -0.120912
    -0.246407
    -0.292141
    0.0990863
    0.331274
    0.133164
    -0.255346
    0.0111306
    -0.180369
    0.0733314
    0.0485543
    -0.000675225
    -0.0391747
    -0.0659976
    -0.0642663
    0.0420131
    0.124276
    0.035692
    -0.024141
    0.232806
    -0.0875989
    -0.0289126
    -0.0256559
    0.0359742
    -0.171378
    -0.0157063
    -0.144515
    -0.0645077
    0.0765323
    -0.0545965
    0.0089937
    0.118871
    -0.227901
    0.117894
    0.0338295
    -0.0148607
    0.0951797
    0.0859544
    -0.0798149
    -0.0333232
    0.114725
    -0.241588
    0.273436
    0.223575
    0.0414235
    0.168515
    0.0783172
    0.0253355
    -0.045372
    0.0127425
    -0.178775
    -0.0632875
    0.0440628
    0.0167294
    0.0563011
    0.0142771
    

# 封装计算类


```python
# %load face_rec.py
import sys
import dlib
import cv2
import os
import glob
import numpy as np


def comparePersonData(data1, data2):
    diff = 0
    # for v1, v2 in data1, data2:
    # diff += (v1 - v2)**2
    for i in range(len(data1)):
        diff += (data1[i] - data2[i])**2
    diff = np.sqrt(diff)
    print(diff)
    if(diff < 0.6):
        print("It's the same person")
    else:
        print("It's not the same person")


def savePersonData(face_rec_class, face_descriptor):
    if face_rec_class.name == None or face_descriptor == None:
        return
    filePath = face_rec_class.dataPath + face_rec_class.name + '.npy'
    vectors = np.array([])
    for i, num in enumerate(face_descriptor):
        vectors = np.append(vectors, num)
        # print(num)
    print('Saving files to :'+filePath)
    np.save(filePath, vectors)
    return vectors


def loadPersonData(face_rec_class, personName):
    if personName == None:
        return
    filePath = face_rec_class.dataPath + personName + '.npy'
    vectors = np.load(filePath)
    print(vectors)
    return vectors


class face_recognition(object):
    def __init__(self):
        self.current_path = os.getcwd()  # 获取当前路径
        self.predictor_path = self.current_path + \
            "\\cnn\\shape_predictor_68_face_landmarks.dat"
        self.face_rec_model_path = self.current_path + \
            "\\cnn\\dlib_face_recognition_resnet_model_v1.dat"
        self.faces_folder_path = self.current_path + "\\faces\\"
        self.dataPath = self.current_path + "\\data\\"
        self.detector = dlib.get_frontal_face_detector()
        self.shape_predictor = dlib.shape_predictor(self.predictor_path)
        self.face_rec_model = dlib.face_recognition_model_v1(
            self.face_rec_model_path)

        self.name = None
        self.img_bgr = None
        self.img_rgb = None
        self.detector = dlib.get_frontal_face_detector()
        self.shape_predictor = dlib.shape_predictor(self.predictor_path)
        self.face_rec_model = dlib.face_recognition_model_v1(
            self.face_rec_model_path)

    def inputPerson(self, name='people', img_path=None):
        if img_path == None:
            print('No file!\n')
            return

        # img_name += self.faces_folder_path + img_name
        self.name = name
        self.img_bgr = cv2.imread(self.current_path+img_path)
        # opencv的bgr格式图片转换成rgb格式
        b, g, r = cv2.split(self.img_bgr)
        self.img_rgb = cv2.merge([r, g, b])

    def create128DVectorSpace(self):
        dets = self.detector(self.img_rgb, 1)
        print("Number of faces detected: {}".format(len(dets)))
        for index, face in enumerate(dets):
            print('face {}; left {}; top {}; right {}; bottom {}'.format(
                index, face.left(), face.top(), face.right(), face.bottom()))

            shape = self.shape_predictor(self.img_rgb, face)
            face_descriptor = self.face_rec_model.compute_face_descriptor(
                self.img_rgb, shape)
            # print(face_descriptor)
            # for i, num in enumerate(face_descriptor):
            #   print(num)
            #   print(type(num))

            return face_descriptor

```

# 测试1


```python
# %load demo5.2.py
import face_rec as fc
face_rec = fc.face_recognition()   # 创建对象
face_rec.inputPerson(name='jobs', img_path='\\faces\\1.jpg')  # name中写第一个人名字，img_name为图片名字，注意要放在faces文件夹中
vector = face_rec.create128DVectorSpace()  # 提取128维向量，是dlib.vector类的对象
person_data1 = fc.savePersonData(face_rec, vector )   # 将提取出的数据保存到data文件夹，为便于操作返回numpy数组，内容还是一样的

# 导入第二张图片，并提取特征向量
face_rec.inputPerson(name='jobs2', img_path='\\faces\\2.jpg')
vector = face_rec.create128DVectorSpace()  # 提取128维向量，是dlib.vector类的对象
person_data2 = fc.savePersonData(face_rec, vector )

# 计算欧式距离，判断是否是同一个人
fc.comparePersonData(person_data1, person_data2)
```

    Number of faces detected: 1
    face 0; left 167; top 67; right 296; bottom 196
    Saving files to :G:\py\dlib\data\jobs.npy
    Number of faces detected: 1
    face 0; left 210; top 110; right 339; bottom 239
    Saving files to :G:\py\dlib\data\jobs2.npy
    0.43092407321591797
    It's the same person
    

# 测试2

如果data文件夹中已经有了模型文件，可以直接导入


```python
# %load demo5.3.py
import face_rec as fc
face_rec = fc.face_recognition()   # 创建对象
person_data1 = fc.loadPersonData(face_rec , 'jobs')   # 创建一个类保存相关信息，后面还要跟上人名，程序会在data文件中查找对应npy文件，比如这里就是'jobs.npy'
person_data2 = fc.loadPersonData(face_rec , 'jobs2')  # 导入第二张图片
fc.comparePersonData(person_data1, person_data2) # 计算欧式距离，判断是否是同一个人
```

    [-0.10090288  0.1072145   0.01279887 -0.07402837  0.0066496   0.02251893
     -0.07831199 -0.09231715  0.19675256 -0.10807184  0.23606203  0.10115325
     -0.20756839 -0.13987558  0.04088261  0.11406779 -0.19969247 -0.0939411
     -0.09967533 -0.11981992  0.0413545  -0.00487671  0.08771094  0.04929478
     -0.13757712 -0.35329792 -0.06079998 -0.17129567  0.00199143 -0.10501849
     -0.08139595  0.00077236 -0.18196386 -0.10702266  0.02165978 -0.02361187
      0.01920501  0.00282077  0.20109291  0.03154382 -0.12579948  0.09426409
      0.02683559  0.23074135  0.27649316  0.08330945 -0.00933838 -0.08498447
      0.12655319 -0.22952324  0.06025456  0.16394231  0.07202258  0.02331806
      0.0970623  -0.19838132 -0.0020905   0.08875141 -0.13795027  0.03113746
      0.00210361 -0.08863088 -0.02868112  0.04059572  0.19180585  0.09500565
     -0.12400492 -0.07089586  0.12965189 -0.02219969  0.02252199  0.0184984
     -0.19471948 -0.21409343 -0.23350216  0.08455663  0.38484025  0.19372645
     -0.19832234  0.03286812 -0.20206457  0.03322642  0.08517597  0.02613049
     -0.07008083 -0.1225555  -0.04213565  0.05716839  0.08312382  0.05342007
     -0.05154483  0.22828871 -0.02528416  0.06432839  0.01616702  0.06495167
     -0.16260289 -0.00856679 -0.17663012 -0.05931918  0.01671621 -0.0178645
      0.05439681  0.14573105 -0.2283798   0.05446758  0.00459545 -0.02577139
      0.01823203  0.07079822 -0.0418243  -0.0374448   0.05113045 -0.25449288
      0.23714027  0.25383428  0.03905518  0.18047769  0.08824719  0.00598699
      0.00265156 -0.02812069 -0.17655382 -0.04851077  0.05448093  0.05907289
      0.07649811  0.00331541]
    [-6.56276643e-02  1.03885919e-01  5.84333241e-02 -9.46767069e-03
      3.13491672e-02  5.99562051e-03 -8.15831944e-02 -5.43527044e-02
      2.10805923e-01 -6.52888864e-02  2.74125129e-01  7.99882561e-02
     -1.57584816e-01 -1.27232060e-01  4.48985100e-02  1.53828233e-01
     -1.50276557e-01 -2.07187757e-02 -1.40310183e-01 -7.66388401e-02
     -6.17932528e-06  1.60488766e-04  7.53390938e-02  1.59993358e-02
     -1.73482582e-01 -3.78838450e-01 -4.94292676e-02 -1.51749954e-01
      1.72043126e-03 -1.52176514e-01 -6.13850914e-02  2.98418920e-03
     -2.27170840e-01 -8.89236853e-02 -4.30098511e-02 -1.98571198e-02
     -6.00264221e-03 -2.56885011e-02  1.16473638e-01  5.01232818e-02
     -1.57516152e-01  1.12152390e-01 -6.76545724e-02  2.34205097e-01
      2.39259332e-01  3.55268009e-02  2.69476771e-02 -4.62278351e-02
      8.57875049e-02 -2.18188748e-01  2.37350836e-02  6.50884509e-02
      4.57599387e-02 -1.63888503e-02  8.86726677e-02 -1.40480235e-01
     -4.69624698e-02  1.32311836e-01 -1.68474615e-01  6.11648224e-02
     -3.11019551e-03 -7.56178722e-02 -6.82836100e-02 -1.50347827e-03
      1.79482132e-01  1.37031436e-01 -1.19423933e-01 -5.55013828e-02
      1.45336628e-01 -3.77072487e-03  6.97276667e-02 -5.66656608e-03
     -1.20911710e-01 -2.46406645e-01 -2.92141348e-01  9.90862995e-02
      3.31274062e-01  1.33164003e-01 -2.55346388e-01  1.11306198e-02
     -1.80368572e-01  7.33314380e-02  4.85542640e-02 -6.75224583e-04
     -3.91747132e-02 -6.59976155e-02 -6.42662570e-02  4.20131311e-02
      1.24276385e-01  3.56920399e-02 -2.41409671e-02  2.32805774e-01
     -8.75988677e-02 -2.89126188e-02 -2.56559085e-02  3.59742008e-02
     -1.71378091e-01 -1.57062970e-02 -1.44514963e-01 -6.45076707e-02
      7.65323266e-02 -5.45964502e-02  8.99369642e-03  1.18871428e-01
     -2.27900982e-01  1.17893651e-01  3.38294841e-02 -1.48606580e-02
      9.51796919e-02  8.59544352e-02 -7.98148885e-02 -3.33232321e-02
      1.14725292e-01 -2.41587505e-01  2.73435503e-01  2.23574698e-01
      4.14235108e-02  1.68515399e-01  7.83172250e-02  2.53354982e-02
     -4.53720056e-02  1.27424598e-02 -1.78775147e-01 -6.32874891e-02
      4.40627933e-02  1.67294331e-02  5.63010573e-02  1.42770782e-02]
    0.43092407321591797
    It's the same person
    

