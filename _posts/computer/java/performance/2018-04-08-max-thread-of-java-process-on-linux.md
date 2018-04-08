---
layout: post
title:  "max thread of java process on linux"
categories: computer
tags:  computer java performance
author: jixianzhao
---

* content
{:toc}


ulimit -Su 250000
echo 600000 > /proc/sys/vm/max_map_count
echo 200000 >  /proc/sys/kernel/pid_max	# seems affectless.
