---
layout: post
title:  "在linux终端连续敲N个ls回车，单核单线程的内核及CPU都干了哪些事"
categories: computer
tags:  computer zhihu linux
author: "in nek"
source: "https://www.zhihu.com/question/314244313/answer/614216534"
---

* content
{:toc}


strace ls

就都知道了。

如果还嫌信息量不够，用qemu运行一个虚拟机，切换到控制台里开trace，直接要求trace到log上，运行一下ls，就什么鬼都出来了。


