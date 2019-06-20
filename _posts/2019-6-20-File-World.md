---
layout: post
title: HDF5 v.s. MATLAB!
header-img: img/
---


![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [ on GitHub.


## Introduction

## Example 1: Huge random matrix
In this example, we create a matrix with the size of 10000 x 10000, and all its elements are produced randomly. Here, we use the function
```python
import numpy as np
a = np.random.rand(10000, 10000)
```

try to store the matrix in the format of .mat using scipy
```python
import scipy.io as sio
sio.savemat('matfile.mat', {'elem1':a})
```
Let's check the size of produced mat file

```ccs
ls -lh matfile.mat
```
`763M`

Now let's come to hdf5, and try to store matrix in the format of .h5

```python
import h5py
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem1', data=a)
```

~~~~
ls -lh h5file.h5
~~~~


we get the size 763M with the command `ls -lh`

```ruby
with h5py.File('h5file2.h5', 'w') as hf:
    hf.create_dataset('elem1', data=a, compression='gzip', compression_opts=9)
```
check the size turn out 720M
```javascript
with h5py.File('h5file3.h5', 'w') as hf:
    hf.create_dataset('elem1', data=a, compression='gzip', compression_opts=4)
```
size 720M, and take longer time
```python
with h5py.File('h5file.h5', 'r') as hf:
    a1 = np.array(hf['elem1'])

with h5py.File('h5file2.h5', 'r') as hf:
    a2 = np.array(hf['elem1'])
    
with h5py.File('h5file3.h5', 'r') as hf:
    a3 = np.array(hf['elem1'])

np.sum(a1-a), np.sum(a2-a), numpy(a3-a)
```
(0.0, 0.0, 0.0)

c = sio.loadmat('matfile.mat')
a4 = c['elem1']

np.sum(a4-a)


## Example 2: Huge sparse matrix

A [sparse matrix](https://en.wikipedia.org/wiki/Sparse_matrix) is a matrix with tons of 'holes'.
dealing with sparse element
```python
a = np.zeros((1000, 1000), 'float32')
sio.savemat('matfile.mat', {'elem1':a})
```
3.9M
```python
with h5py.File('h5file.h5', 'w') as hf:
    hf.create_dataset('elem1', data=a)
```
3.9M
```python
with h5py.File('h5file2.h5', 'w') as hf:
    hf.create_dataset('elem1', data=a, compression='gzip', compression_opts=9)
```
21K



