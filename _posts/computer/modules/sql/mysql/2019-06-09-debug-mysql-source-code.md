---
layout: post
title: "debug mysql source code"
categories: computer
tags:  computer sql mysql stackoverflow debug_mysql
author: "web"
source: ""
---

* content
{:toc}


**base on:**  
mysql version 5.7,   
debian 9,   
linux kernel 4.19.0-0.bpo.4-amd64  

# jxz operation

## prepare

I don't know if this neccesary. caused by "collect2: fatal error: cannot find 'ld'".  

	sudo apt-get install build-essential  

Acturally, I fixed it by download boost_1_59_0.tar.gz.  

## don't build in source.



## compile

>
cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/data/debug/mysql5/install -DMYSQL_DATADIR=/data/debug/mysql5/data -DSYSCONFDIR=/data/debug/mysql5/etc -DMYSQL_UNIX_ADDR=/data/debug/mysql5/mysql.sock -DWITH_DEBUG=1 -DDOWNLOAD_BOOST=1 -DWITH_BOOST=/data/code/github/mysql5-dep -DDOWNLOAD_BOOST_TIMEOUT=60000 -G "CodeBlocks - Unix Makefiles" /data/code/github/mysql-server5  

>
make -j 4  

>
\# try to run it without sudo at free  
make install -j 4  

>
\# init data  
./mysqld --initialize-insecure --user=root --datadir=/data/debug/mysql5/data  

>
\# startup  
./mysqld --defaults-file=/data/debug/mysql5/etc/my.cnf  

>
\# debug config  
-DCMAKE_INSTALL_PREFIX=/data/debug/mysql5/install  
-DMYSQL_DATADIR=/data/debug/mysql5/data  
-DSYSCONFDIR=/data/debug/mysql5/etc  
-DMYSQL_UNIX_ADDR=/data/debug/mysql5/mysql.sock  
-DWITH_DEBUG=1  
-DWITH_BOOST=/data/code/github/mysql5-dep  

## initialize data

copy from other database instance.

## config

### my.cnf
>
[client]  
user=root  
password=123456  
>
[mysqld]  
user=tomato  
innodb_file_per_table=1  
datadir=/data/debug/mysql5/data  


### apparmor

	vim /etc/apparmor.d/usr.sbin.mysqld5

>
	/data/debug/mysql5/install/bin/mysqld {  
		/lib/** rmwk,  
		/usr/lib/** rmwk,  
		/proc/*/status r,  
		/sys/devices/system/node/ r,  
		/sys/devices/system/node/node*/meminfo r,  
		/sys/devices/system/node/*/* r,  
		/sys/devices/system/node/* r,  
		/** rwmk,  
		/data/debug/mysql5/** r,  
		/data/debug/mysql5/** rwk,  
		/tmp/** rwk,  
	}

### check apparmor log

	usermod -a -G systemd-journal mysql

##  start

	./mysqld_safe --skip-grant-tables --socket=/data/debug/mysql5/mysql.sock


# config



## Unable to start MySQL on Ubuntu: AVC apparmor="DENIED" operation="open"

source:[link](https://support.plesk.com/hc/en-us/articles/360004185293-Unable-to-start-MySQL-on-Ubuntu-AVC-apparmor-DENIED-operation-open-)


### Symptoms

MySQL on Ubuntu fails to start. The following error messages are shown in the output of the `journalctl -xe` command:
>
AVC apparmor="DENIED" operation="open" profile="/usr/sbin/mysqld" name="**/proc/**666999**/status**" pid=666999 comm="mysqld" requested\_mask="r" denied\_mask="r" fsuid=124 ouid=124  
AVC apparmor="DENIED" operation="open" profile="/usr/sbin/mysqld" name="**/sys/devices/system/node/**" pid=666999 comm="mysqld" requested\_mask="r" denied\_mask="r" fsuid=124 ouid=0  
audit: type=1400 audit(): apparmor="DENIED" operation="open" profile="/usr/sbin/mysqld" name="**/proc/**666999**/status**" pid=666999 comm="mysqld" requested\_mask="r" denied\_mask="r" fsuid=124 ouid=124  
audit: type=1400 audit(): apparmor="DENIED" operation="open" profile="/usr/sbin/mysqld" name="**/sys/devices/system/node/**" pid=666999 comm="mysqld" requested\_mask="r" denied\_mask="r" fsuid=124 ouid=0

### Cause

AppArmor is not properly configured.

### Resolution

Add permissions for the objects reported in the lines that start with '`name=`' in the output to the file `/etc/apparmor.d/usr.sbin.mysqld`.

In this example, it is required to add `r` permissions to `/proc/*/status` and `/sys/devices/system/node/`. The paths may be different depending on the error messages.

1.  Connect the server via [SSH](https://support.plesk.com/hc/en-us/articles/115000172834).
    
2.  Open the file `/etc/apparmor.d/usr.sbin.mysqld` in a text editor (for example, [vi editor](https://support.plesk.com/hc/en-us/articles/360001084114)) and add the lines below at the end of the `/usr/sbin/mysqld` section at the end of the file:
	
	>
	/usr/sbin/mysqld {  
		...  
		/proc/\*/status r,  
		/sys/devices/system/node/ r,  
		/sys/devices/system/node/node\*/meminfo r,  
		/sys/devices/system/node/\*/\* r,  
		/sys/devices/system/node/\* r,  
		...  
	}
    
3.  Reload AppArmor configuration for MySQL service:
    
	\# apparmor\_parser -r /etc/apparmor.d/usr.sbin.mysqld
    
4.  Start MySQL server:
    
	\# service mysql start
    
    **Note:** In case MySQL fails to start again, check the output of `journalctl -xe` for new error messages.
    

### Additional Information

*   [16.04 - MySQL won't start because of AppArmor? - Ask Ubuntu](https://askubuntu.com/questions/916009/mysql-wont-start-because-of-apparmor)



## AppArmor

source: [official manual](https://wiki.ubuntu.com/AppArmor)

### Introduction

AppArmor is a Mandatory Access Control (MAC) system which is a kernel (LSM) enhancement to confine programs to a limited set of resources. AppArmor's security model is to bind access control attributes to programs rather than to users. AppArmor confinement is provided via profiles loaded into the kernel, typically on boot. AppArmor profiles can be in one of two modes: enforcement and complain. Profiles loaded in enforcement mode will result in enforcement of the policy defined in the profile as well as reporting policy violation attempts (either via syslog or auditd). Profiles in complain mode will not enforce policy but instead report policy violation attempts.

AppArmor differs from some other MAC systems on Linux: it is path-based, it allows mixing of enforcement and complain mode profiles, it uses include files to ease development, and it has a far lower barrier to entry than other popular MAC systems.

AppArmor is an established technology first seen in Immunix and later integrated into Ubuntu, Novell/SUSE, and Mandriva. Core AppArmor functionality is in the mainline Linux kernel from 2.6.36 onwards; work is ongoing by AppArmor, Ubuntu and other developers to merge additional AppArmor functionality into the mainline kernel.

### Example profile

From /etc/apparmor.d/usr.sbin.tcpdump on Ubuntu 9.04:

>
	#include <tunables/global>
	/usr/sbin/tcpdump {
	  #include <abstractions/base>
	  #include <abstractions/nameservice>
	  #include <abstractions/user-tmp>
	  capability net\_raw,
	  capability setuid,
	  capability setgid,
	  capability dac\_override,
	  network raw,
	  network packet,
	  # for -D
	  capability sys\_module,
	  @{PROC}/bus/usb/ r,
	  @{PROC}/bus/usb/\*\* r,
	  # for -F and -w
	  audit deny @{HOME}/.\* mrwkl,
	  audit deny @{HOME}/.\*/ rw,
	  audit deny @{HOME}/.\*/\*\* mrwkl,
	  audit deny @{HOME}/bin/ rw,
	  audit deny @{HOME}/bin/\*\* mrwkl,
	  @{HOME}/ r,
	  @{HOME}/\*\* rw,
	  /usr/sbin/tcpdump r,
	}

The above profile for tcpdump demonstrates several properties of AppArmor:

*   profiles are simple text files
*   comments are supported in the profile
*   absolute paths as well as file globbing can be used when specifying file access
*   various access controls for files are present. From the profile we see 'r' (read), 'w' (write), 'm' (memory map as executable), 'k' (file locking), and 'l' (creation hard links). There are others not demonstrated in this profile, including (but not limited to) 'ix' (execute and inherit this profile), 'Px' (execute under another profile, after cleaning the environment), and 'Ux' (execute unconfined, after cleaning the environment)
*   access controls for capabilities are present
*   access controls for networking are present
*   specificity in rule matching, ie the most specific rule matches (eg access to @{HOME}/bin/bad.sh is denied with auditing due to 'audit deny @{HOME}/bin/\*\* mrwkl,' even though general access to @{HOME} is permitted with '@{HOME}/\*\* rw,')
    
*   include files are supported to ease development and simplify profiles (ie #include <abstractions/base>, #include <abstractions/nameservice>, #include <abstractions/user-tmp>)
    
*   variables can be defined and manipulated outside the profile (#include <tunables/global> with @{PROC} and @{HOME})
    
*   AppArmor profiles are easy to read and audit
    

Please see [More information](/AppArmor#More.2520information) below for full details on updating and developing profiles as well as instructions using AppArmor.

### AppArmor in Ubuntu

AppArmor support was first introduced in Ubuntu 7.04, and is turned on by default in Ubuntu 7.10 and later. AppArmor confinement in Ubuntu is application specific with profiles available for specific binaries. With each release, [more and more profiles](/SecurityTeam/KnowledgeBase/AppArmorProfiles) are shipped by default, with [more planned](/SecurityTeam/Roadmap#AppArmor_Confinement).

If a profile is not available for an application, users may create a profile and add it to /etc/apparmor.d. If a profile is not defined for a particular binary, the binary is not confined. See [More information](/AppArmor#More_information) for details.


# first login

## mysqld_safe — MySQL Server Startup Script

[official manual](https://dev.mysql.com/doc/refman/5.5/en/mysqld-safe.html)  

>
--socket=path


## How to find out the MySQL root password

source: [stackoverflow](https://stackoverflow.com/questions/10895163/how-to-find-out-the-mysql-root-password) 

On ubuntu I did the following:

    sudo service mysql stop
    sudo mysqld_safe --skip-grant-tables --skip-syslog --skip-networking

Then run mysql in a new terminal:

    mysql -u root

And run the following queries to change the password:

    UPDATE mysql.user SET authentication_string=PASSWORD('password') WHERE User='root';
    FLUSH PRIVILEGES;

_In MySQL 5.7, the password field in mysql.user table field was removed, now the field name is 'authentication\_string'._

Quit the mysql safe mode and start mysql service by:

    mysqladmin shutdown
    sudo service mysql start



## How to Fix MariaDB Plugin ‘unix_socket’ is not loaded Error

source: [link](https://www.linuxbabe.com/mariadb/plugin-unix_socket-is-not-loaded-2)

**Last Updated: June 13, 2017**

The Error “Plugin ‘unix\_socket’ is not loaded” is commonly seen on Ubuntu 15.04/15.10/16.04 and any derivative distributions such as Linux Mint 18. This tutorial shows how to fix it.

### What is the Unix\_Socket Plugin?

The Unix\_Socket authentication plugin, which allows users to use OS credentials to connect to [MariaDB](https://www.linuxbabe.com/mariadb/install-mariadb-10-1-ubuntu14-04-15-10) via Unix socket, is first supported in MariaDB 5.2.0. This plugin is not installed by default.

Log into MariaDB monitor.

	mysql -u root -p

Then install the Unix\_Socket plugin with this command:

>
MariaDB \[(none)\]>  install plugin unix\_socket soname 'auth\_socket';

My Ubuntu system has a user named `linuxbabe`, so I create a MariaDB user `linuxbabe` identified via the unix\_socket plugin.

>
MariaDB \[(none)\]> create user linuxbabe identified via unix\_socket;

Exit out of MariaDB monitor.

>
MariaDB \[(none)\]> quit

And now I can log into MariaDB monitor as user linuxbabe without typing password because I already logged into Ubuntu system as linuxbabe.

![mariadb unix_socket plugin](/image/mysql/2019-06-09-debug-mysql-source-code-1-mariadb-unix_socket-plugin.png)

That’s how Unix\_Socket authentication plugin works.

### Fix Plugin ‘unix\_socket’ is not loaded Error

The Unix\_Socket authentication plugin only works when your Linux OS and MariaDB have a user account with the same username.

Your Linux OS has a root user. MariaDB also has a root user. So sometimes, when you try to log into MariaDB monitor as root user, MariaDB may authenticate you via the Unix\_Socket plugin but this plugin is not installed by default. So you see `Plugin 'unix_socket' is not loaded` Error.

Another authentication plugin is `mysql_native_password`. MariaDB uses this plugin to authenticate a user who is created with this command:

>
create user demo\_user@localhost identified by password 'secret\_password';

To fix the above error, we can tell MariaDB to use `mysql_native_password` plugin to authenticate root user.

First stop MariaDB. If you have installed MariaDB from Ubuntu repository, use this command to stop it.

	sudo systemctl stop mysql

If you [have installed MariaDB from MariaDB repository](/image/mysql/2019-06-09-debug-mysql-source-code-2-mariadb-unix_socket-authentication-plugin.png), use the following command to stop it.

	sudo systemctl stop mariadb

Then start MariaDB with `--skip-grant-tables` option which bypass user authentication.

	sudo mysqld\_safe --skip-grant-tables &

Next, log into MariaDB monitor as root.

	mysql -u root

Enter the following SQL statement to check which authentication plugin is used for root.

>
MariaDB \[(none)\]> select Host,User,plugin from mysql.user where User='root';

![Plugin 'unix_socket' is not loaded](https://www.linuxbabe.com/wp-content/uploads/2016/06/mariadb-unix_socket-authentication-plugin.png)

You might see it’s using unix\_socket plugin. To change it to mysql\_native\_password plugin, execute this command:

>
MariaDB \[(none)\]> update mysql.user set plugin='mysql\_native\_password';

If you forgot the MariaDB root user password, you can also change the root password now with the following command:

>
MariaDB \[(none)\]> update mysql.user set password=PASSWORD("newpassword") where User='root';

Exit MariaDB monitor.

>
flush privileges;
quit;

Stop mysqld\_safe

	sudo kill -9 $(pgrep mysql)

Start MariaDB again.

	sudo systemctl start mysql     
	
or   

	sudo systemctl start mariadb

Now you can use normal password to login.

mysql -u root -p

I hope this article helped you to fix Plugin ‘unix\_socket’ is not loaded error. Comments, questions or suggestions are always welcome. If you found this post useful, ? please share it with your friends on social media! Stay tuned for more tutorials.





## How to Solve “ERROR 1524 (HY000): Plugin ‘unix_socket’ is not loaded ” MySQL error on Debian / Ubuntu

source: [link](https://computingforgeeks.com/how-to-solve-error-1524-hy000-plugin-unix_socket-is-not-loaded-mysql-error-on-debian-ubuntu/)

**(Last Updated On: April 21, 2019)**

I recently encountered this error “**ERROR 1524 (HY000): Plugin ‘unix\_socket’ is not loaded**” on my Debian 9 server while trying to authenticate to MariaDB database as a root user. After some googling, I found the workaround for this error is to start MySQL service with `mysqld_safe` and reset the root password.

### Step 1: Stop mysql service

Stop MySQL service.

	$ sudo systemctl stop mysql  
OR  

	$ sudo /etc/init.d/mysql stop  

### Step 2: Start mysql with mysqld\_safe

Then start mysql service with `mysqld_safe` and option `--skip-grant-tables` and keep in running it the background.

>
$ **sudo mysqld\_safe --skip-grant-tables &**  
\[1\] 8197

### Step 3: Reset root user password

Open MySQL terminal console.

>
$ **mysql -uroot**  
Welcome to the MariaDB monitor.  Commands end with ; or \\g.  
Your MariaDB connection id is 8  
Server version: 10.3.14-MariaDB-1:10.3.14+maria~stretch mariadb.org binary distribution  
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.  
Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.  
  >
MariaDB \[(none)\]>

You should get to MySQL terminal without password authentication. Now switch to `mysql` database.

>
MariaDB \[(none)\]> **USE mysql;**  
Reading table information for completion of table and column names  
You can turn off this feature to get a quicker startup with -A  
>
Database changed

Reset root user password.

>
MariaDB \[mysql\]> **update user set password=PASSWORD("NewRootPassword") where User='root';**  
Query OK, 0 rows affected (0.002 sec)  
Rows matched: 1  Changed: 0  Warnings: 0

Replace `NewRootPassword` with your new password for root user.

Also set Authentication plugin to native.

>
MariaDB \[mysql\]> **UPDATE USER SET plugin="mysql\_native\_password";**  
Query OK, 2 rows affected (0.001 sec)  
Rows matched: 2  Changed: 2  Warnings: 0

Close your Database session.

>
MariaDB \[mysql\]> **quit;**  
Bye

### Step 4: Start MySQL Service normally

Restart mysql service in a standard manner. But first stop the service.

	$ sudo systemctl stop mysql  
	
OR  

	$ /etc/init.d/mysql stop

Ensure no other process is running.

	ps aux | grep mysql

Start mysql.

	sudo systemctl start mysql

Test access as root user.

$ **mysql -u root -p**  
Enter password: <Enter Password Set>  
Welcome to the MariaDB monitor.  Commands end with ; or \\g.  
Your MariaDB connection id is 8  
Server version: 10.3.14-MariaDB-1:10.3.14+maria~stretch mariadb.org binary distribution  
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.  
Type 'help;' or '\\h' for help. Type '\\c' to clear the current input statement.  
MariaDB \[(none)\]> **QUIT**  
Bye



## How To Create a New User and Grant Permissions in MySQL

source: [link](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql)

>
mysql> CREATE USER 'tomato'@'localhost' IDENTIFIED BY '123456';
>
mysql> GRANT ALL PRIVILEGES ON *.* TO 'tomato'@'localhost';
>
FLUSH PRIVILEGES;



## How to Connect to MySQL Without Root Password on Terminal

source: [link](https://www.tecmint.com/connect-to-mysql-without-root-password/)

**helpless**

Normally while installing **MySQL/MariaDB** database server on Linux, it’s recommended to set a MySQL root user password to secure it, and this password is required to access the database server with root user privileges.

**Suggested Read:** [How to Install and Secure MariaDB 10 in CentOS 7](https://www.tecmint.com/install-mariadb-in-centos-7/)

In this guide, we will show you how to connect and run MySQL commands without entering a password (mysql passwordless root login) on the Linux terminal.

### How to Set MySQL Root Password

In case you have freshly installed the MySQL/MariaDB server, then it doesn’t require any password to connect to it as root user. To secure it, set the MySQL/MariaDB password for root user with the following command.

Note that this command is just one of the many [MySQL (Mysqladmin) Commands for Database Administration](https://www.tecmint.com/mysqladmin-commands-for-database-administration-in-linux/) in Linux.

\# mysqladmin -u root password **YOURNEWPASSWORD**

### How to Connect or Run MySQL Without Root Password

To run MySQL commands without entering password on the terminal, you can store your user and password in the `~/.my.cnf` user specific configuration file in user’s home directory as described below.

Now create the config file `~/.my.cnf` and add configurations below in it (remember to replace **mysqluser** and **mysqlpasswd** with your own values).

>
\[mysql\]
user=user
password=password

Save and close the file. Then set the suitable permissions on it, to make it only readable and writable by you.

	# chmod 0600 .my.cnf

Once you have set user and password in the Mysql configuration file, from now on when you run mysql commands such as mysql, mysqladmin etc, they will read the mysqluser and mysqlpasswd from the above file.

	# mysql 
	# mysql -u root 

![Run Mysql- Commands Without Root Password](/image/mysql/2019-06-09-debug-mysql-source-code-3-Run-Mysql-Without-Root-Password.png)


In this guide, we showed how to run MySQL commands without entering root password on the terminal. If you have any queries or thoughts to share, do hit us up via the feedback form below.


## Column count of mysql.user is wrong. Expected 42, found 44. The table is probably corrupted

source: [link](https://stackoverflow.com/questions/43846950/column-count-of-mysql-user-is-wrong-expected-42-found-44-the-table-is-probabl)

### question

Currently I'm using the newest version of ISPConfig 3. Today I wanted to add a db and user. It didn't work. Then I tried it on PHPmyadmin and it didn't work.

When I tried to add a user in PHPMyadmin Users Panel I received the following error message:

>
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '* TO 'test'@'localhost'' at line 1

The output from /var/log/mysql/error.log:

>
[ERROR] Column count of mysql.user is wrong. Expected 42, found 44. The table is probably corrupted

Mysql Version: 5.5.55-0+deb8u1 PHPMyadmin Version: 4:4.2.12-2+deb8u2

Debian Linux 8

### answer

Migrating from mariadb 10 to mysql 5.6 saw similar issues. The error message I received, was slightly different than the others listed on this page... which, of course, means it required a different solution. Upon attempting to modify a user record I received the following error:

>
Column count of mysql.user is wrong. Expected 43, found 46. The table is probably corrupted

Some of the advice above helped frame the problem. After taking a look at a similar server (to the mysql 5.6 one), I compared the fields in the both the "corrupted" user table (from the mariadb 10 mysql.users table) & the "functional" user table in the other mysql 5.6 mysql.users table.

I removed the three problematic fields using the mysql cli & the following commands:

	mysql -u root -p
	
>
use mysql;  
alter table mysql.user drop column default_role;  
alter table mysql.user drop column max_statement_time;  
alter table mysql.user drop column password_expired;  
quit  

Problem resolved!

### in my case jxz

>
mysql> CREATE USER 'tomato'@'localhost' IDENTIFIED BY '123456';
ERROR 1805 (HY000): Column count of mysql.user is wrong. Expected 45, found 48. The table is probably corrupted


## How to Run MySQL as a Normal User

source: [official manual](https://dev.mysql.com/doc/refman/5.5/en/changing-mysql-user.html)

**helpless**

### 6.1.5 How to Run MySQL as a Normal User

On Windows, you can run the server as a Windows service using a normal user account.

On Linux, for installations performed using a MySQL repository, RPM packages, or Debian packages, the MySQL server [**mysqld**](mysqld.html "4.3.1 mysqld — The MySQL Server") should be started by the local `mysql` operating system user. Starting by another operating system user is not supported by the init scripts that are included as part of the installation.

On Unix (or Linux for installations performed using `tar` or `tar.gz` packages) , the MySQL server [**mysqld**](mysqld.html "4.3.1 mysqld — The MySQL Server") can be started and run by any user. However, you should avoid running the server as the Unix `root` user for security reasons. To change [**mysqld**](mysqld.html "4.3.1 mysqld — The MySQL Server") to run as a normal unprivileged Unix user _`user_name`_, you must do the following:

1.  Stop the server if it is running (use [**mysqladmin shutdown**](mysqladmin.html "4.5.2 mysqladmin — Client for Administering a MySQL Server")).
    
2.  Change the database directories and files so that _`user_name`_ has privileges to read and write files in them (you might need to do this as the Unix `root` user):
    
    Press CTRL+C to copy
    
     
    
    `shell> chown -R _user_name_ _/path/to/mysql/datadir_`
    
    If you do not do this, the server will not be able to access databases or tables when it runs as _`user_name`_.
    
    If directories or files within the MySQL data directory are symbolic links, `chown -R` might not follow symbolic links for you. If it does not, you will also need to follow those links and change the directories and files they point to.
    
3.  Start the server as user _`user_name`_. Another alternative is to start [**mysqld**](mysqld.html "4.3.1 mysqld — The MySQL Server") as the Unix `root` user and use the [``--user=_`user_name`_``](server-options.html#option_mysqld_user) option. [**mysqld**](mysqld.html "4.3.1 mysqld — The MySQL Server") starts, then switches to run as the Unix user _`user_name`_ before accepting any connections.
    
4.  To start the server as the given user automatically at system startup time, specify the user name by adding a `user` option to the `[mysqld]` group of the `/etc/my.cnf` option file or the `my.cnf` option file in the server's data directory. For example:
    
    Press CTRL+C to copy
    
     
    
    `[mysqld]
    user=_user_name_`
    

If your Unix machine itself is not secured, you should assign passwords to the MySQL `root` accounts in the grant tables. Otherwise, any user with a login account on that machine can run the [**mysql**](mysql.html "4.5.1 mysql — The MySQL Command-Line Client") client with a [`--user=root`](mysql-command-options.html#option_mysql_user) option and perform any operation. (It is a good idea to assign passwords to MySQL accounts in any case, but especially so when other login accounts exist on the server host.) See [Section 2.10.4, “Securing the Initial MySQL Accounts”](default-privileges.html "2.10.4 Securing the Initial MySQL Accounts").

### jxz:

[Warning] One can only use the --user switch if running as root.  




