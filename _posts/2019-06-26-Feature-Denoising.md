---
layout: post
title: \[CVPR2019\] Feature Denoising for Improving Adversarial Robustness 后感
subtitle: 浅谈怎么讲好一个故事
header-img: img/post-bg-trap.jpg 
catalog: true
tags: CVPR19 Denoising
---

## 前言
今天某Y想分享一篇来自自何恺明大神组的CVPR2019的文章，并通过该文章谈下关于如何讲好一个故事的个人感受。

## 背景介绍
这篇文章首先描述了一个很有意思的现象:在遭受<strong>对抗攻击（adversarial attack）</strong>时，卷积神经网络（CNN）会对图像的理解产生偏差。

<blockquote>对抗攻击是指对图像加入小幅度的扰动，来诱使判别器对其作出错误的判断。</blockquote>

通过下图我们可以看到，即便是在人眼感受上很小的扰动，依然会致使网络做出错误的判断（将“电子钟”误检成“加热器”）。这就不得不使得人们开始思考:

* 现实世界中可能存在这样的潜在威胁。（想象一个基于DL的人脸识别系统能够轻易地被戏弄。阔怕）

* 网络的运算处理和人脑有着很大的区别。（人脑还是很厉害的！）
g


