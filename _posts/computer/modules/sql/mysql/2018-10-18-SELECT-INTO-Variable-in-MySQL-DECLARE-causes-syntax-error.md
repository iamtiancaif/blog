---
layout: post
title: "SELECT INTO Variable in MySQL DECLARE causes syntax error"
categories: computer
tags: computer copy mysql
author: "web"
source: "https://stackoverflow.com/questions/3075147/select-into-variable-in-mysql-declare-causes-syntax-error"
---

* content
{:toc}


I ran into this same issue, but I think I know what's causing the confusion. If you use MySql Query Analyzer, you can do this just fine:

    SELECT myvalue 
    INTO @myvar 
    FROM mytable 
    WHERE anothervalue = 1;

However, if you put that same query in MySql Workbench, it will throw a syntax error. I don't know why they would be different, but they are. To work around the problem in MySql Workbench, you can rewrite the query like this:

    SELECT @myvar:=myvalue
    FROM mytable
    WHERE anothervalue = 1;








