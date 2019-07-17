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

这个问题通常是由<strong>图像</strong>大小引起的（文字档案通常size很小）。为了解决这个问题，最简单的方法就是利用在线图像工具进行压缩，尤其是tif，jpg的图像，压缩率可以高达<strong>80%</strong>！这里推荐一个常用的[在线工具](https://compressjpeg.com/zh/)，可以很方便地<strong>批量</strong>处理以及下载。另外他们家的其他各类转换工具也很好用，切完全免费！

## .bbl文件缺失
使用overleaf的过程中，参考文献通常会写在```.bib```文件里。再在正文中用如下命令调用
```ccs
\bibliographystyle{IEEEtran}
\bibliography{reference.bib}
```

然而将overleaf上的文件直接打包下载再上传arxiv的时候会报出如下错误
<img src="/img/post-oa-bbl-error.jpg" width="700"/>

解决方案很简单，这里需要利用到overleaf提供的<strong>submit</strong>功能，替你编译并生成```.bbl```文件。submit界面中选择arxiv并```Download project ZIP with submission files (e.g. .bbl)```。界面如下所示
<img src="/img/post-oa-overleaf-submit.jpg" width="700"/>

此时一切大功告成！！？？才怪！接下来是<strong>最重要的两步</strong>！！

* 将```.bbl```文件的名称改为```.bib```文件的名称
* 在正文中将上面的引用文献代码改成如下所示！
```ccs
\input{reference.bbl}
\bibliographystyle{IEEEtran}
\bibliography{reference}
```

此时再上传arxiv，我们就可以顺利通过文件检查😄😄😄

## pgfplots包引起的谜之bug

之所以称这个是谜之bug，是因为在overleaf上可以顺利加载pgfplots包且编译顺利，但是在arxiv上则会失败。引起的原因大概是arxiv上的tex编译系统与overleaf的latex编译系统的存在兼容性问题。

话不多说，解决方案是这样的：在调用pgfplots包之后，```\begin{document}```之前需添加下面这段命令，注意！<strong>arxiv只支持到1.14版本</strong>！
```
\pgfplotsset{compat=1.14}
```

此后bug顺利解决！😆😆😆

## 路径名一致
若```.zip```文件是由文件夹打包而来，那么需要注意上传后的路径是从文件夹名起始记录。因而单独再额外上传文件的时候可能会导致路径不一致，从而文件无法被读取。

## 小结
希望某Y在这的踩坑总结可以帮助到被坑到的大家~祝大家paper都能被顺利接收~😉😉😉


