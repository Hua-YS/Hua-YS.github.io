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

<strong>报错！</strong>呀哈，果然不能直接用（[然而并不是这样的](#cv2resize)）。于是某Y在网上搜索后找到了这样一个看着非常相似的函数<strong>np.resize</strong>！于是赶紧使用看看
```python
np.resize(x, (h/2, w/2, c))
```

\usepackage{pgfplots}
\pgfplotsset{compat=1.14}

\input{reference.bbl}
\bibliographystyle{IEEEtran}
\bibliography{reference}
