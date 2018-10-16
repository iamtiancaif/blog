---
layout: post
title: "Spring @Transactional注解不回滚不起作用无效"
categories: computer
tags:  computer copy java spring transaction
author: "web"
source: "https://www.cnblogs.com/huacw/p/8075143.html"
---

* content
{:toc}


这几天在项目里面发现我使用@Transactional之后，抛了异常居然不回滚。后来终于找到了原因。   
如果你也出现了这种情况，可以从下面开始排查。

### 一、特性

先来了解一下@Transactional注解的特性吧，可以更好排查问题

1. service类标签(一般不建议在接口上)上添加@Transactional，可以将整个类纳入spring事务管理，在每个业务方法执行时都会开启一个事务，不过这些事务采用相同的管理方式。

2. @Transactional 注解只能应用到 public 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，不过事务设置不会起作用。

3. 默认情况下，spring会对unchecked异常进行事务回滚；如果是checked异常则不回滚。   
辣么什么是checked异常，什么是unchecked异常？

java里面将派生于Error或者RuntimeException（比如空指针，1/0）的异常称为unchecked异常，
  
其他继承自java.lang.Exception得异常统称为Checked Exception，如IOException、TimeoutException等

再通俗一点：你写代码出现的空指针等异常，会被回滚，文件读写，网络出问题，spring就没法回滚了。

4。 只读事务：   
@Transactional(propagation=Propagation.NOT\_SUPPORTED,readOnly=true)   
只读标志只在事务启动时应用，否则即使配置也会被忽略。   
启动事务会增加线程开销，数据库因共享读取而锁定(具体跟数据库类型和事务隔离级别有关)。通常情况下，仅是读取数据时，不必设置只读事务而增加额外的系统开销。

### 二、事务传播模式

Propagation枚举了多种事务传播模式，部分列举如下：

1\. REQUIRED(默认模式)：业务方法需要在一个容器里运行。如果方法运行时，已经处在一个事务中，那么加入到这个事务，否则自己新建一个新的事务。

2\. NOT\_SUPPORTED：声明方法不需要事务。如果方法没有关联到一个事务，容器不会为他开启事务，如果方法在一个事务中被调用，该事务会被挂起，调用结束后，原先的事务会恢复执行。

3\. REQUIRESNEW：不管是否存在事务，该方法总汇为自己发起一个新的事务。如果方法已经运行在一个事务中，则原有事务挂起，新的事务被创建。

4\. MANDATORY：该方法只能在一个已经存在的事务中执行，业务方法不能发起自己的事务。如果在没有事务的环境下被调用，容器抛出例外。

5\. SUPPORTS：该方法在某个事务范围内被调用，则方法成为该事务的一部分。如果方法在该事务范围外被调用，该方法就在没有事务的环境下执行。

6\. NEVER：该方法绝对不能在事务范围内执行。如果在就抛例外。只有该方法没有关联到任何事务，才正常执行。

7\. NESTED：如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。

三、解决Transactional注解不回滚
----------------------

1\. 检查你方法是不是public的。

2\. 你的异常类型是不是unchecked异常。  
如果我想check异常也想回滚怎么办，注解上面写明异常类型即可。

@Transactional(rollbackFor=Exception.class)

类似的还有norollbackFor，自定义不回滚的异常。

3\. 数据库引擎要支持事务，如果是mysql，注意表要使用支持事务的引擎，比如innodb，如果是myisam，事务是不起作用的。

4\. 是否开启了对注解的解析

<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>

5\. spring是否扫描到你这个包，如下是扫描到org.test下面的包

<context:component-scan base-package="org.test" ></context:component-scan>

以上是想到的可能出现的情况，以后有再添加。












