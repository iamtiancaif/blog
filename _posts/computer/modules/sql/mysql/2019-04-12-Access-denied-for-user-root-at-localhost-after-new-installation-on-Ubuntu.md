---
layout: post
title: "Access denied for user root at localhost after new installation on Ubuntu"
categories: computer
tags:  computer sql mysql stackoverflow
author: "web"
source: "https://stackoverflow.com/questions/28068155/access-denied-for-user-rootlocalhost-using-password-yes-after-new-instal"
---

* content
{:toc}


**TL;DR:** To access newer versions of mysql/mariadb after as the root user, after a new install, you need to be in a root shell (ie `sudo mysql -u root`, or `mysql -u root` inside a shell started by `su -` or `sudo -i` first)

* * *

Having just done the same upgrade, on Ubuntu, I had the same issue.

What was odd was that

    sudo /usr/bin/mysql_secure_installation

Would accept my password, and allow me to set it, but I couldn't log in as `root` via the `mysql` client

I had to start mariadb with

    sudo mysqld_safe --skip-grant-tables

to get access as root, whilst all the other users could still access fine.

Looking at the `mysql.user` table I noticed for root the `plugin` column is set to `unix_socket` whereas all other users it is set to 'mysql\_native\_password'. A quick look at this page: [https://mariadb.com/kb/en/mariadb/unix\_socket-authentication-plugin/](https://mariadb.com/kb/en/mariadb/unix_socket-authentication-plugin/) explains that the Unix Socket enables logging in by matching `uid` of the process running the client with that of the user in the `mysql.user` table. In other words to access mariadb as `root` you have to be logged in as root.

Sure enough restarting my mariadb daemon with authentication required I can login as root with

    sudo mysql -u root -p

or

    sudo su -
    mysql -u root -p

Having done this I thought about how to access without having to do the sudo, which is just a matter of running these mysql queries

    GRANT ALL PRIVILEGES on *.* to 'root'@'localhost' IDENTIFIED BY '<password>';
    FLUSH PRIVILEGES;

or

    UPDATE mysql.user SET plugin = 'mysql_native_password' WHERE user = 'root' AND plugin = 'unix_socket';
    FLUSH PRIVILEGES;

Then restarting mariadb:

    sudo service mysql restart

And voila I had access from my personal account vi mysql -u root -p

**PLEASE NOTE THAT DOING THIS IS REDUCING SECURITY** Presumably the MariaDB developers have opted to have root access work like this for a good reason.

Thinking about it I'm quite happy to have to `sudo mysql -u root -p` so I'm switching back to that, but I thought I'd post my solution as I couldn't find one elsewhere.  


