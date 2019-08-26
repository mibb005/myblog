---
title: jupyter用法
date: 2019-08-26 10:49:07
tags:
 - jupyter
---

# 将本地的.py文件load到jupyter的一个cell中


```python
# %load test.py
print('test')
```

    test
    

# jupyter的cell可以作为unix command使用


```python
!python --version 
```

    Python 3.7.0b2
    


```python
!python test.py
```

    test
    

# 从网络load代码到jupyter


```python
# %load https://matplotlib.org/examples/color/color_cycle_demo.html
```

# 获取current working directory


```python
current_path = %pwd 
print(current_path)
```

    G:\tensorflow\Jupyter
    

# 运行代码


```python
print("Hello, World!")
```

    Hello, World!
    


```python
for letter in 'Python':     # 第一个实例
   print('当前字母 :', letter)
 
fruits = ['banana', 'apple',  'mango']
for fruit in fruits:        # 第二个实例
   print('当前水果 :', fruit)
 
print("Good bye!")
```

    当前字母 : P
    当前字母 : y
    当前字母 : t
    当前字母 : h
    当前字母 : o
    当前字母 : n
    当前水果 : banana
    当前水果 : apple
    当前水果 : mango
    Good bye!
    

# 使用Matplotlib绘图


```python
%matplotlib inline
```


```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(20)
y = x**2

plt.plot(x, y)
```




    [<matplotlib.lines.Line2D at 0x1d3e0edc3c8>]




![png](output_15_1.png)



```python

```
