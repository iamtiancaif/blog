---
layout: post
title:  "install software on centos"
categories: computer
tags:  computer linux centos stackexchange
author: web
source: "https://unix.stackexchange.com/questions/60924/how-to-install-iftop"
ignore: true
---

* content
{:toc}



For CentOS 7 (fully updated today), install from EPEL repository

	yum install epel-release -y
	yum install iftop -y

Not 100% sure, but I think earlier versions require downloading the EPEL repository and installing manually using rpm, then you can just "yum install iftop" (as per other answers)




