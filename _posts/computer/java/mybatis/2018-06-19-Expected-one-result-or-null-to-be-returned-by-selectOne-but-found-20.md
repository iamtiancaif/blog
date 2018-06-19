---
layout: post
title:  "Expected one result (or null) to be returned by selectOne(), but found: 20"
categories: computer
tags:  computer java mybatis copy
author: web
source: "https://blog.csdn.net/u010453363/article/details/46621319"
---

* content
{:toc}


报错：
---

	Exception in thread "main" org.mybatis.spring.MyBatisSystemException: nested exception is org.apache.ibatis.exceptions.TooManyResultsException: Expected one result (or null) to be returned by selectOne(), but found: 20


  
疑问：
------

myabtis的xml配置bean数据库等均无错误，当 select返回单个结果集时，不会报错，多个结果集时则报错。

mybatis和spring整合后，dao中的接口

**\[java\]** [view plain](https://blog.csdn.net/u010453363/article/details/46621319# "view plain") [copy](https://blog.csdn.net/u010453363/article/details/46621319# "copy")

1.  public Student getUser(Student userStudent);  

  
会自动选择selectOne（）方法，由于只能返回一个对象的结果，当返回到多个对象的结果集时报错。getUser是一个student类的bean无法接受多个结果集，则改用List就可以接受多个结果集

  

解决：
---

将dao中的

	public Student getUser(Student userStudent);  

  
改为

	import java.util.List;  
	public List<Student> getUser(Student userStudent);  

故障排除















Expected one result (or null) to be returned by selectOne(), but found: 20
