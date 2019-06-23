---
layout: post
title: resize的陷阱
subtitle: np.resize v.s. cv2.resize
header-img: img/post-bg-trap.jpg 
catalog: true
tags: Data_Processing
---

## 前言
最近，某Y在对矩阵做resize操作时不小心踩了个大坑。情况大概是这样的：某Y希望利用双线性插值将一个三维的矩阵在spatial维度上进行缩小。通常如果channel维度小于等于3的话（即图像），某Y会毫不犹豫地利用<strong>cv2.resize</strong>。然而这次的矩阵，channel维度远高于3！这就一下子唬住某Y了，怎么办呢？cv2.resize通常是用来对图像矩阵进行操作的，能直接用于眼下的情况吗?带着这样的疑惑，某Y颤颤地尝试了下面这个</strong>非常愚蠢</strong>的代码
```python
cv2.resize(x, (h/2, w/2, c))
```

<strong>报错！</strong>呀哈，果然不能直接用（[然而并不是这样的](#cv2resize)）。于是某Y在网上搜索后找到了这样一个看着非常相似的函数<strong>np.resize</strong>！于是赶紧使用看看
```python
np.resize(x, (h/2, w/2, c))
```

<strong>没有报错！而且size也确实按照预期进行改变了！</strong>由于矩阵较大且每个元素都是浮点小数难以检查，某Y就直接用了这个函数生成的数据进行网络训练。然而训练的结果一塌糊涂！一盆冷水直接浇醒了某Y，也使得某Y认真检视起这两个函数并做了以下的实验。结论可跳至[这里](#conclusion)。

## Experiment 1: np.resize到底干了什么？
为了清楚地看到每个元素的数值变化，我们先利用numpy创建了如下测试矩阵
```python
a = np.array([[1, 2, 3, 4], [6, 5, 4, 3]])
a = a[:, :, np.newaxis]
a = np.repeat(a, 6, axis=-1)
```

随后利用np.shape可以得到该矩阵各个维度的信息
```python
h, w, c = np.shape(a)
print h, w, c
```

可以看到这是一个2&#xd7;4&#xd7;6的矩阵。接下来我们利用np.resize使这个矩阵的长宽减半，同时保证channel维度不变
```python
b = np.resize(a, (int(h/2), int(w/2), c))
np.shape(b)
```

可以看到，我们成功的得到了一个各维度（1&#xd7;2&#xd7;6）都符合我们预期的矩阵，那么这个矩阵的内部是怎样的呢？
```python
b
array([[[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2]]])
```

原矩阵是这个样子的
```python
a
array([[[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2],
        [3, 3, 3, 3, 3, 3],
        [4, 4, 4, 4, 4, 4]],

       [[6, 6, 6, 6, 6, 6],
        [5, 5, 5, 5, 5, 5],
        [4, 4, 4, 4, 4, 4],
        [3, 3, 3, 3, 3, 3]]])
```

<strong>坑爹呢这！！！玩呢？！</strong>通过检查函数文档可以看到这样一句描述
<blockquote><p>Warning: This functionality does not consider axes separately, i.e. it does not apply interpolation/extrapolation. It fills the return array with the required number of elements</p></blockquote>

好吧，你赢了，怪我一时兴奋没有仔细读文档。。。那么我们再试着将长宽double

```python
b = np.resize(a, (int(h*2), int(w*2), c))
b
array([[[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2],
        [3, 3, 3, 3, 3, 3],
        [4, 4, 4, 4, 4, 4],
        [6, 6, 6, 6, 6, 6],
        [5, 5, 5, 5, 5, 5],
        [4, 4, 4, 4, 4, 4],
        [3, 3, 3, 3, 3, 3]],

       [[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2],
        [3, 3, 3, 3, 3, 3],
        [4, 4, 4, 4, 4, 4],
        [6, 6, 6, 6, 6, 6],
        [5, 5, 5, 5, 5, 5],
        [4, 4, 4, 4, 4, 4],
        [3, 3, 3, 3, 3, 3]],

       [[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2],
        [3, 3, 3, 3, 3, 3],
        [4, 4, 4, 4, 4, 4],
        [6, 6, 6, 6, 6, 6],
        [5, 5, 5, 5, 5, 5],
        [4, 4, 4, 4, 4, 4],
        [3, 3, 3, 3, 3, 3]],

       [[1, 1, 1, 1, 1, 1],
        [2, 2, 2, 2, 2, 2],
        [3, 3, 3, 3, 3, 3],
        [4, 4, 4, 4, 4, 4],
        [6, 6, 6, 6, 6, 6],
        [5, 5, 5, 5, 5, 5],
        [4, 4, 4, 4, 4, 4],
        [3, 3, 3, 3, 3, 3]]])

```

恩，只是单纯的<strong>copy</strong>和<strong>delete</strong>。为什么会有这么奇怪的函数！那么问题回来了，该怎么实现利用双线性插值沿着矩阵的spatial维度缩放呢？

## <a name="cv2resize"></a>Experiment 2: 还是该用cv2.resize！
对！还是该用<strong>cv2.resize！</strong>只需要把愚蠢的代码改正即可（第一行不加会报错！）
```python
a = np.float32(a)
c = cv2.resize(a, (int(h/2), int(w/2)))
np.shape(c)
```

然而我们这是会发现输出的是一个<strong>2&#xd7;1&#xd7;6</strong>的矩阵！！？？通过查看资料后发现此处需要将<strong>w</strong>与<strong>h</strong>对调位置，即
```python
c = cv2.resize(a, (int(w/2), int(h/2)))
np.shape(c)
```

这样便可以得到我们想要的各个维度分别做差值的矩阵，同时将矩阵长宽double也是一样的操作
```python
c = cv2.resize(a, (int(w*2), int(h*2)))
np.shape(c)
```

此时我们随机输出某一维度进行观察
```python
a[:, :, 3]
array([[1., 2., 3., 4.],
       [6., 5., 4., 3.]], dtype=float32)
c[:, :, 3]
array([[1.   , 1.25 , 1.75 , 2.25 , 2.75 , 3.25 , 3.75 , 4.   ],
       [2.25 , 2.375, 2.625, 2.875, 3.125, 3.375, 3.625, 3.75 ],
       [4.75 , 4.625, 4.375, 4.125, 3.875, 3.625, 3.375, 3.25 ],
       [6.   , 5.75 , 5.25 , 4.75 , 4.25 , 3.75 , 3.25 , 3.   ]],
      dtype=float32)
```

明显可以看到，c是由a差值得到！

## <a name="conclusion"></a>结论
cv2.resize还是很厉害的！但是在使用时一定要注意以下两点：
* 输入矩阵数据中元素的<strong>数据类型</strong>需符合cv2的要求（e.g.，float32）
* 变换后的<strong>长宽</strong>对应原始的宽长


## 后话
唉。改代码，重新生成数据咯。。。





