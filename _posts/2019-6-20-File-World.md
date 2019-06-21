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

可以看到系统输出的文件尺寸为<strong>763M</strong>。接下来我们尝试用h5py来存储该矩阵。

### 1.2: .h5 &#xd7; h5py
[HDF](https://zh.wikipedia.org/wiki/HDF)的全称是Hierarchical Data Format（层级数据格式）。这是一种专门用来存储和管理大型数据的文件格式。其最初是由美国国家超级计算应用中心（NCSA）开发，现在则由非营利社团HDF Group管理运营。在这里我们提到的.h5是该格式的第五代，也就是HDF5对应的文件名后缀。python中的相关工具包为h5py。

这里我们首先导入h5py并以其中的[File](http://docs.h5py.org/en/stable/high/file.html) object对象来存储和管理数据。此处<em>h5file.h5</em>为待创建的存储文件。和使用savemt相同，这里我们需要先定义存储对象为<em>elem<\em>, 再将矩阵以`data=`的形式赋予该变量。存
```python
import h5py
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem', data=a)
```

```ccs
ls -lh h5file.h5
```


we get the size 763M with the command `ls -lh`

```python
with h5py.File('h5file2.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=9)
```
check the size turn out 720M
```python
with h5py.File('h5file3.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=4)
```
size 720M, and take longer time
```python
with h5py.File('h5file.h5', 'r') as hf:
    a1 = np.array(hf['elem'])

with h5py.File('h5file2.h5', 'r') as hf:
    a2 = np.array(hf['elem'])
    
with h5py.File('h5file3.h5', 'r') as hf:
    a3 = np.array(hf['elem'])

np.sum(a1-a), np.sum(a2-a), numpy(a3-a)
```
(0.0, 0.0, 0.0)

c = sio.loadmat('matfile.mat')
a4 = c['elem1']

np.sum(a4-a)


## Example 2: Huge sparse matrix

A [sparse matrix](https://en.wikipedia.org/wiki/Sparse_matrix) is a matrix with tons of 'holes'.
dealing with sparse element

### 2.1: scipy &#xd7; .mat
```python
a = np.zeros((1000, 1000), 'float32')
sio.savemat('matfile.mat', {'elem':a})
```
3.9M
### 2.2: h5py &#xd7; .h5
```python
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem', data=a)
```
3.9M
```python
with h5py.File('h5file2.h5', 'w') as hf:
    hf.create_dataset('elem', data=a, compression='gzip', compression_opts=9)
```
21K



