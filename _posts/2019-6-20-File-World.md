---
layout: post
title: Saving a large matrix as .mat or .h5?
header-img: img/
---

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [ on GitHub.


## Introduction

we want to compare the compression efficiency and information loss of h5py and scipy. 

## Example 1: Huge random matrix
In this example, we test `scipy` and `h5py` with a huge 10000 x 10000 random matrix.
```python
import numpy as np
a = np.random.rand(10000, 10000)
```
### 1.1: scipy &#xd7; .mat
First, we use `scipy` to store the matrix in the format of <strong>.mat</strong>
```python
import scipy.io as sio
sio.savemat('matfile.mat', {'elem':a})
```

After the process completed, let's check the size of the produced .mat file
```ccs
ls -lh matfile.mat
```
The size turns out `763M`, which is not a small number.

### 1.2: h5py &#xd7; .h5
Now let's come to hdf5, and try to store matrix in the format of .h5

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



