---
layout: post
title: \[CVPR2019\] Feature Denoising for Improving Adversarial Robustness 后感
subtitle: 浅谈怎么讲好一个故事
header-img: img/post-bg-trap.jpg 
catalog: true
tags: CVPR19 Denoising
---

## 前言
今天某Y想分享一篇来自自何恺明大神组的CVPR2019的文章，并通过该文章谈下关于如何讲好一个故事的个人感受。在故事线上，某Y做了一些调整以方便讲述。

## 背景介绍
这篇文章首先描述了一个很有意思的现象:在遭受<strong>对抗攻击（adversarial attack）</strong> 时，卷积神经网络（CNN）会对图像的理解产生偏差。

<blockquote>对抗攻击是指对图像加入小幅度的扰动，来诱使判别器对其作出错误的判断。值得注意的是，此处的<em>小幅度</em>是指人眼难以觉察到的程度。</blockquote>

通过下图我们可以看到，即便是在人眼感受上很小的扰动，依然会致使网络做出错误的判断（将“电子钟”误检成“加热器”）。这就不得不使得人们开始思考:

* 现实世界中可能存在这样的潜在<strong>威胁</strong>。（想象一个基于DL的人脸识别系统能够轻易地被戏弄。阔怕）
* 网络的运算处理和人脑有着很大的<strong>区别</strong>。（人脑还是很厉害的！）

值得注意的是，此处的“小幅度”是指人眼难以觉察到的程度。

为了更清楚的描绘这个现象，作者给了更多的对抗攻击干扰网络特征提取的例子。其中最右的示例尤为明显，非常少量的噪声严重干扰了特征的提取。针对这样的现象，作者对feature noise进行了分析，并指出

* 对扰动的约束仅在图像的pixel级别存在，<strong>而在feature层面上并没有任何的约束</strong>。
* <strong>扰动会随着在网络中的传递而增大</strong>！

因此每经过一个层，扰动都会随之增大，直至淹没true signal，使得网络无法被正确地激活。

为了解决这个问题，作者在文章中提出在feature 上进行denoising的想法，并设计了相应的denoising 模块。








