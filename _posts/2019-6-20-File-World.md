---
layout: post
title: 大型矩阵究竟该如何存储?
subtitle: .mat与.h5之间的抉择
header-img: img/
---

最近，某Y在创建模型的训练数据集时遇到了一个非常头疼的问题－－生成的训练数据占用了过大的存储空间致使磁盘爆满！可能因为以前都是在体量较小的数据库上进行研究，故而第一次碰到这个问题的某Y是非常崩溃的。相信被下面这句话支配的恐惧感（尤其是在没有管理员权限的服务器上）很多和某Y一样的小白都深有感触吧（哭）
```ccs
Unable to create xxx: Disk quota exceeded
```
然而也正是这句话促使了某Y认真研究起数据的存储格式以及不同存储格式之间的效率问题。

通常，由于MATLAB是科研工作者的常用工具，<strong>.mat</strong>也成了最常见的数据存储格式之一。然而，当遇到大型矩阵时，.mat究竟是不是一个最有效的存储格式呢？有没有什么更有效的存储格式呢？下面某Y通过两组实验来对比下<strong>.mat</strong>和<strong>.h5</strong>的存储效率：

## Experiment 1: 大型随机矩阵
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

存储完毕后我们进入到数据存储位置并使用如下命令检查文件大小
```ccs
ls -lh matfile.mat
```

可以看到系统输出的文件大小为<strong>763M</strong>。


接下来我们尝试用h5py来存储该矩阵。

### 1.2: .h5 &#xd7; h5py
[HDF](https://zh.wikipedia.org/wiki/HDF)的全称是Hierarchical Data Format（层级数据格式）。这是一种专门用来存储和管理大型数据的文件格式。其最初是由美国国家超级计算应用中心（NCSA）开发，现在则由非营利社团HDF Group管理运营。在这里我们提到的.h5是该格式的第五代，也就是HDF5对应的文件名后缀。python中的相关工具包为h5py。

这里我们首先导入h5py并以其中的[File](http://docs.h5py.org/en/stable/high/file.html) object对象来存储和管理数据。此处<em>h5file.h5</em>为待创建的存储文件。和使用savemt相同，这里我们需要先定义存储对象为<em>elem</em>, 再将矩阵以`data=`的形式赋予该变量
```python
import h5py
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem', data=a)
```

存储完毕后我们进入到数据存储位置并使用如下命令检查文件大小
```ccs
ls -lh h5file.h5
```

文件大小依旧为<strong>763M</strong>!!某Y看到这个的时候内心是绝望的，难道说好的高效牛p都是骗人的吗？难道我的磁盘空间没救了吗？带着这个疑问，某Y又做了详细的功课并发现一个惊人的功能－－[<strong>压缩</strong>](http://docs.h5py.org/en/stable/high/dataset.html?highlight=compression)。是的，h5py还提供了`compression`和`compression_opts`这两个变量可供设置。而在compression中，共有三种压缩方式可供选择：`gzip`，`lzf`，`szip`！这可不得了，我们赶紧试验下利用压缩后文件会有怎样的变化，这里我们选择文档中推荐的压缩方法gzip，同时`compression_opts`我们尝试了默认值4以及最高值9
```python
with h5py.File('h5file_com4.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=4)
    
with h5py.File('h5file_com9.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=9)
```

压缩后的文件体积缩小了！两种压缩效率得到的文件均为<strong>720M</strong>！另外值得一提的是使用默认值4的时候，存储速度明显慢于最高值9。这时候可能有人就会质疑了：<strong>会不会有信息损失呢？</strong>为了回答这个疑问，我们继续做了下面实验，从存储文件中将我们的随机矩阵读入python并与原始矩阵对比
```python
with h5py.File('h5file_com4.h5', 'r') as hf:
    a_com4 = np.array(hf['elem'])

with h5py.File('h5file_com9.h5', 'r') as hf:
    a_com9 = np.array(hf['elem'])
    
np.sum(a_com4-a), np.sum(a_com9-a)
```

对比结果是
```python
(0.0, 0.0)
```

不多说了，<strong>h5py用起来好嘛！</strong>

等等，某Y突然想到，自己的数据集中，样本大部分为大型的稀疏矩阵，那么存储成.h5格式是不是会更节约空间呢？为此我们设计了第二个实验来验证我们的想法

## Experiment 2: 大型稀疏矩阵
[稀疏矩阵](https://en.wikipedia.org/wiki/Sparse_matrix)指的是大部分元素为零的矩阵。因此在存储此类矩阵时，不同的存储方式都会采取一定的措施大幅度压缩文件体积。这里我们在极端情况，即矩阵元素全部为0，上进行不同存储方式之间的比较
```python
a = np.zeros((10000, 10000))
```

### 2.1: scipy &#xd7; .mat
首先我们用scipy来对矩阵进行存储
```python
sio.savemat('matfile.mat', {'elem':a})
```

存储完成后，生成的文件大小为<strong>763M<strong>, 8.7s
### 2.2: h5py &#xd7; .h5
```python
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem', data=a)
```
3.9M
```python
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=9)
```
1.3M 5.1s



