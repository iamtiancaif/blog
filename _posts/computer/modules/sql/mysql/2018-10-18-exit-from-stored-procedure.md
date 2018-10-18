---
layout: post
title: "exit from stored procedure"
categories: computer
tags: computer copy mysql
author: "web"
source: "https://stackoverflow.com/questions/6260157/mysql-how-to-quit-exit-from-stored-procedure"
---

* content
{:toc}


	CREATE PROCEDURE SP_Reporting(IN tablename VARCHAR(20))
	proc_label:BEGIN
		 IF tablename IS NULL THEN
			  LEAVE proc_label;
		 END IF;
	
		 #proceed the code
	END;



reply
------

Great! You even point out that the END proc_label; syntax (shown in most official MySQL examples) is not needed. (this is a great way to comment out a stored proc without having to scroll to the bottom to put */ in place)   




