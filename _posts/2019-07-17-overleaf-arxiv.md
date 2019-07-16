---
layout: post
title: overleaf to arxiv的踩坑细节
subtitle: 搞定bbl & pgfplots
header-img: img/post-bg-trap.jpg 
catalog: true
tags: Writing
---

## 前言
为了把overleaf上写的东西上传到arxiv上，真的是折腾的要死。。。记录下踩坑细节，为以后的自己和其它人留个记录

## 文件总size
这个是第一道门槛，通常overleaf上写文章并不需要考虑所有文件的总大小，但是在上传arxiv的时候却有着极其严格的限制--<strong>10MB</strong>！！！一旦超过了这个size，不管多优秀的文章都无法上传！

这个问题通常是由<strong>图像</strong>大小引起的（文字档案通常size很小）。为了解决这个问题，最简单的方法就是利用在线图像工具进行压缩，尤其是tif，jpg的图像，压缩率可以高达<strong>80%<strong>！这里推荐一个常用的[在线工具](https://compressjpeg.com/zh/)，可以很方便地<strong>批量</strong>处理以及下载。另外他们家的其他各类转换工具也很好用，切完全免费！


<strong>报错！</strong>呀哈，果然不能直接用（[然而并不是这样的](#cv2resize)）。于是某Y在网上搜索后找到了这样一个看着非常相似的函数<strong>np.resize</strong>！于是赶紧使用看看
```python
np.resize(x, (h/2, w/2, c))
```

\usepackage{pgfplots}
\pgfplotsset{compat=1.14}

\input{reference.bbl}
\bibliographystyle{IEEEtran}
\bibliography{reference}
