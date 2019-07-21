---
layout: post
title: "怎么样使用CLion调试分析MySQL Server"
categories: computer
tags:  computer sql mysql debug_mysql
author: "web"
source: "https://my.oschina.net/u/222608/blog/1511382"
---

* content
{:toc}

由于在写MySQL日志订阅服务时候，需要确定在什么event之后保存position，所以就开始研究MySQL的源码，刚开始采用最原始的打印输出的方式去调试，然后每次改完编译运行，效率好低，让我很绝望，然后我花了些时间研究下怎么使用CLion Debug MySQL。

获取源码

    git clone https://github.com/mysql/mysql-server
    

编译安装初始化数据库

    cd mysql-server
    
    cmake \
    -DCMAKE_INSTALL_PREFIX=/path/mysql/install \
    -DMYSQL_DATADIR=/path/mysql/data \
    -DSYSCONFDIR=/path/mysql/etc \
    -DMYSQL_UNIX_ADDR=/path/mysql/mysql.sock \
    -DWITH_DEBUG=1  \
    -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/path/mysql-server/ -DDOWNLOAD_BOOST_TIMEOUT=60000
    
    make -j 4 
    
    make install -j 4
    
    mysqld --initialize-insecure --user=root --datadir=/path/mysql/data
    
    

启动MySQL，测试下是否安装成功 

    /path/install/bin/mysqld --defaults-file=/path/mysql/etc/my.cnf
    

使用CLion新建工程并打开源码目录之后，设置CLion ![设置CLion CMake选项](/image/mysql/2019-06-09-how-to-debug-mysql-server-with-clion-1.png "设置CLion CMake选项")

CMake Options和你编译安装时的选项一致

    -DCMAKE_INSTALL_PREFIX=/path/mysql/install 
    -DMYSQL_DATADIR=/path/mysql/data 
    -DSYSCONFDIR=/path/mysql/etc 
    -DMYSQL_UNIX_ADDR=/path/mysql/mysql.sock 
    -DWITH_DEBUG=1  
    

然后在CLion里，Reload CMake Project

![Reload CMake Project](/image/mysql/2019-06-09-how-to-debug-mysql-server-with-clion-2.png "Reload CMake Project")

在Run/Debug列表里就可以看到很多选项了

![Run/Debug](/image/mysql/2019-06-09-how-to-debug-mysql-server-with-clion-3.png "Run/Debug")

找到mysqld配置下启动参数 ![mysqld配置下启动参数](/image/mysql/2019-06-09-how-to-debug-mysql-server-with-clion-4.png "mysqld配置下启动参数")

    mysqld --defaults-file=/path/mysql/etc/my.cnf
    

然后以Debug模式启动，看下成功的效果 ![Debug模式启动](/image/mysql/2019-06-09-how-to-debug-mysql-server-with-clion-5.png "Debug模式启动")

学习MySQL源码的文档

    https://dev.mysql.com/doc/internals/en/
    

可以找到想要学习的功能的源码位置，不至于没头苍蝇，比如主从同步功能（replication） 

    https://dev.mysql.com/doc/internals/en/replication-source-code-files.html
    

更多架构、PHP、GO相关踩坑实践技巧请关注我的公众号：PHP架构师



