---
layout: post
title:  "Updating MySQL primary key"
categories: computer
tags:  computer copy stackoverflow mysql
author: "web"
source: "https://stackoverflow.com/questions/2341576/updating-mysql-primary-key"
---

* content
{:toc}


Next time, use a single "alter table" statement to update the primary key.

    alter table xx drop primary key, add primary key(k1, k2, k3);

To fix things:

    create table fixit (user_2, user_1, type, timestamp, n, primary key( user_2, user_1, type) );
    lock table fixit write, user_interactions u write, user_interactions write;
    
    insert into fixit 
    select user_2, user_1, type, max(timestamp), count(*) n from user_interactions u 
    group by user_2, user_1, type
    having n > 1;
    
    delete u from user_interactions u, fixit 
    where fixit.user_2 = u.user_2 
      and fixit.user_1 = u.user_1 
      and fixit.type = u.type 
      and fixit.timestamp != u.timestamp;
    
    alter table user_interactions add primary key (user_2, user_1, type );
    
    unlock tables;

The lock should stop further updates coming in while your are doing this. How long this takes obviously depends on the size of your table.

The main problem is if you have some duplicates with the same timestamp.













