---
layout: post
title: "tidb in practice"
categories: computer
tags:  computer tidb practice
author: "joseph"
source: ""
---

* content
{:toc}
# [How to determine the right TiDB and TiDB-Ansible version](https://stackoverflow.com/questions/50984802/how-to-determine-the-right-tidb-and-tidb-ansible-version)

You can view the version number using the following two methods:

- `select tidb_version()`
- `tidb-server -V`





# [Couldn’t connect to Docker daemon at http+docker://localhost - is it running?](https://blog.csdn.net/ken1583096683/article/details/83104124 )

我使用的是ubuntu16.04
原因在于当前用户没有docker的使用权限，需要使用sudo

或者将当前用户加入到docker用户组，这个请参考我写的另一篇文章：
非root用户没有权限使用docker

这个问题可以参考github上的issue：https://github.com/docker/compose/issues/4181




# [tidb使用坑记录](https://www.cnblogs.com/linn/p/8459327.html)

date: 2018-02-22 16:31 

1、对硬盘要求很高，没上SSD硬盘的不建议使用

2、不支持分区，删除数据是个大坑。

解决方案：set @@session.tidb_batch_delete=1; 

3、插入数据太大也会报错

解决方案：set @@session.tidb_batch_insert=1; 

4、删除表数据时不支持别名

delete from 表名 表别名 where 表别名.col = '1'  会报错

5、内存使用有问题，GO语言导致不知道回收机制什么时候运作。内存使用过多会导致TIDB当机（这点完全不像MYSQL）

测试情况是，32G内存，在10分钟后才回收一半。

6、数据写入的时候，tidb压力很大, tikv的CPU也占用很高

7、不支持GBK

8、不支持存储过程

9、列数支持太少，只支持100列，和oralce/mysql的1000列少太多（Oracle 最大列数为 1000；MySQL对于每个表具有4096个列的硬限制, 其中InnoDB每个表的限制为1017列, 最大行大小限制为65,535字节）

  

外面文章的一些建议

```
3TiKV＋3PD＋2TiDB

在有了 TiSpark 之后，我们便利用 TiSpark 将中间表缓存为 Spark 的内存表，只需要将最后的数据落地回 TiDB，再执行 Merge 操作即可，这样省掉了很多中间数据的落地，大大节省了很多脚本执行的时间

在查询速度解决之后，我们发现脚本中会有很多针对中间表 update 和 delete 的语句。目前 TiSpark 暂时不支持 update 和 delete 的操作（和 TiSpark 作者沟通，后续会考虑支持这两个操作），
我们便尝试了两种方案，一部分执行类似于 Hive，采用 insert into 一张新表的方式来解决；另外一部分，我们引入了 Spark 中的 Snappydata 作为一部分内存表存储，
在 Snappydata 中进行 update 和 delete，以达到想要的目的。因为都是 Spark 的项目，因此在融合两个项目的时候还是比较轻松的。

最后，关于实时的调度工具，目前我们是和离线调度一起进行调度，这也带来了一些问题，每次脚本都会初始化一些 Spark 参数等，这也相当耗时。在未来，我们打算采用 Spark Streaming 作为调度工具，
每次执行完成之后记录时间戳，Spark Streaming 只需监控时间戳变化即可，能够避免多次初始化的耗时，通过 Spark 监控，我们也能够清楚的看到任务的延迟和一些状态，这一部分将在未来进行测试。
```



