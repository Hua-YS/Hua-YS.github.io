---
layout: post
title: resize的陷阱
subtitle: np.resize v.s. cv2.resize
header-img: img/post-bg-trap.jpg 
catalog: true
tags: Processing
---

## 前言
最近，某Y在对矩阵做resize操作时不小心踩了个大坑。情况大概是这样的：某Y希望利用双线性插值将一个三维的矩阵在spatial维度上进行缩小。通常如果channel维度小于等于3的话（即图像），某Y会毫不犹豫地利用<strong>cv2.resize</strong>。然而这次的矩阵，channel维度远高于3！这就一下子唬住某Y了，怎么办呢？cv2.resize通常是用来对图像矩阵进行操作的，能直接用于眼下的情况吗?带着这样的疑惑，某Y颤颤地尝试了下面这个非常愚蠢的代码
```python
cv2.resize(x, (h/2, w/2, c))
```

报错！呀哈，果然不能直接用（然而并不是这样的）。于是某Y在网上搜索后找到了这样一个看着非常相似的函数<strong>np.resize</strong>！于是赶紧使用看看
```python
np.resize(x, (h/2, w/2, c))
```

没有报错！而且size也确实按照预期进行改变了！由于矩阵较大且每个元素都是浮点小数难以检查，某Y就直接用了这个函数生成的数据进行网络训练。然而训练的结果一塌糊涂！一盆冷水直接浇醒了某Y，也使得某Y认真检视起这两个函数并做了以下的实验

## Experiment 1: np.resize到底干了什么？
在这个实验中， 我们着重比较的是两种不同存储格式对于大型随机矩阵的存储效率。首先，我们通过numpy创建一个10000 &#xd7; 10000的大型随机矩阵
```python
import numpy as np
a = np.random.rand(10000, 10000)
```
随后，我们将分别利用python中提供的.mat存储工具包[scipy](https://www.scipy.org/)以及.h5的存储工具[h5py](https://www.h5py.org/)来存储这个大型随机矩阵。

### 1.1: .mat &#xd7; scipy
这里我们首先导入scipy并利用其中的[savemat](https://docs.scipy.org/doc/scipy/reference/generated/scipy.io.savemat.html)函数进行存储。存储的文件名为<em>matfile.mat</em>。注意存储对象需编码成字典形式才能存储`{'variable_name'：data}`。此处我们将矩阵存储为变量<em>elem</em>
```python
import scipy.io as sio
sio.savemat('matfile.mat', {'elem':a})
```

