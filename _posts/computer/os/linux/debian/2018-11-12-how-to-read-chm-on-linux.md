---
layout: post
title:  "how to read chm on linux"
categories: computer
tags:  computer linux debian
author: web
source: "https://blog.csdn.net/chenhezhuyan/article/details/23098129"
---

* content
{:toc}


最近工作一直在buntu系统上，有时候需要查看chm文件，但是chm文件是windows的产物，如何在linux查看呢。

有两种办法.

第一种方法：安装firefox的chmreader插件，使用火狐浏览器打开。  
>
1、从http://sourceforge.net/projects/chmreader下载chmreader. 
2、从firefox中打开下载的xpi文件. 
3、重新启动firefox就安装了chmreader插件. 
4、打开chm文件(通过file中的open CHM files)

第二种方法：（我的系统是ubuntu系统）执行 sudo apt-get install chmsee，安装完后，找一个chm文件，点击右键属性，点击属性中的打开方式，选中chmsee作为默认值。这时候打开chm文件可能是乱码，然后点击工具栏的配置选项，字体选择东亚-中国  
>
https://www.cnblogs.com/wangkongming/p/4476794.html  
(64位)    sudo dpkg -i chmsee_1.3.0-2ubuntu2_amd64.deb  安装软件包,下载地址:http://kr.archive.ubuntu.com/ubuntu/pool/universe/c/chmsee/chmsee_1.3.0-2ubuntu2_amd64.deb  Ubuntu的官方软件库放心下载.  
(32位)    sudo dpkg -i chmsee_1.3.0-2ubuntu2_i386.deb 安装软件包,下载地址:http://kr.archive.ubuntu.com/ubuntu/pool/universe/c/chmsee/chmsee_1.3.0-2ubuntu2_i386.deb Ubuntu的官方软件库放心下载.  
如果遇到未安装依赖包，就先执行 sudo apt-get install -f  




jxz
----

chmsee有bug，  
在火狐搜chm能找到chm reader，然后安装。  



