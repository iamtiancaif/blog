---
layout: post
title:  "Windows10内置Linux子系统初体验"
categories: computer
tags:  computer linux windows
author: web
source: "https://www.jianshu.com/p/bc38ed12da1d"
ignore: true
---

* content
{:toc}


pic1

WSL

### 前言

* * *

前段时间，机子上的win10又偷偷摸摸升级到了一周年正式版，比较无奈。不过之前听闻这个版本已经支持内置的linux子系统，于是就怀着好奇心试玩了一把。虽然期间遇到了很多问题，但总体来说这个子系统体验还不错，在这里就分享一些关键步骤和遇到的问题，剩下的大家自己折腾吧。。

### 启用linux子系统

* * *

pic2  

设置(16215版之后不再需要开发人员模式)

pic3

Windows功能

pic4

  

安装ing...

> 1.  系统升级到一周年正式版及以上(1607)
>     
> 2.  依次在 `设置` - `更新与安全` - `针对开发人员` 选项中，启用"**开发人员模式**"
>     
> 3.  在资源管理器中打开 `控制面板\所有控制面板项\程序和功能` , 打开 `启用或关闭 Windows功能` , 勾选 `适用于Linux的Windows子系统(Beta)`
>     
> 4.  重启电脑
>     
> 5.  命令行运行 `lxrun /install /y` 开始安装  
>     安装速度取决于网络情况，下载的文件在 `%localappdata%\lxss` 目录下 `lxss.tar.gz` (181M)，解压后大概500M, `rootfs` 目录即为子系统根目录。
>     
> 6.  命令行运行 `bash` 进入**Ubuntu**  
>     默认使用的 `root` 帐号登录，通过指令 `passwd` 设置密码。
>     
> 
> *   **注：本文脚本均在root帐号下操作，因此建议使用root帐号**
> 
> 7.  毕竟爱折腾，难免会把子系统环境(lxss目录)玩坏掉，因此干正事前最好先备份下以便快速还原。注意，不要直接右键复制或者打包，可能会导致文件权限丢失的。  
>     `xcopy %localappdata%\lxss %localappdata%\lxss.bak /E`
> 8.  当然，如果你比较任性也可以不执行上一步的备份操作，通过命令行运行 `lxrun /uninstall /full` 轻松卸载子系统，重复上面的步骤即可重装，不过要注意下载速度时好时坏哦。

通过上面的步骤，已经启用了win10自带的linux子系统(WSL)，感觉逼格提升了不少。当然，怎么能满足于此呢，接下来就要做一些环境的配置和进一步的挖掘。

### 更换数据源([参考](http://blog.csdn.net/sunnyliqian/article/details/50179915))

* * *

在**Ubuntu**下我们可以通过 [apt-get 命令](http://www.cnblogs.com/xiangzi888/archive/2012/03/18/2405075.html) 很方便的安装/卸载软件，由于默认的软件包仓库是位于国外的，安装软件的时候就可能遇到各种网络问题或者下载到的一些资源不完整，因此就需要切换数据源为国内的镜像站点来改善。

    # 1.备份原来的数据源配置文件
    cp /etc/apt/sources.list /etc/apt/sources.list_backup
    # 2.编辑数据源配置文件
    vi /etc/apt/sources.list
    

在这里我使用的是阿里云的数据源：

    deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse
    

    # 3.更新配置
    apt-get update
    

注：14986版之后更新了内核，第三方的镜像站可能找不到软件包资源，需要切换回官方的源。经测试[中科大的源](http://mirrors.ustc.edu.cn/help/ubuntu.html)可用

    deb https://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
    deb https://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
    

### 与 Windows 通讯

* * *

目前子系统与 Windows 之间通过以下两种方式进行通讯

> 1.  通过 `tcp` 协议进行通讯（简单点说就是用网络，端口都是通的）
> 2.  通过 `/mnt/【盘符】/目录` 的方式访问Windows目录  
>     试过在Windows的资源管理器中直接对子系统环境目录下的文件所做的修改不能被子系统所识别，因此需要在bash下进行操作。

[在任何情况下，请勿使用Windows应用程序，工具，脚本，控制台等创建或修改Linux文件](https://blogs.msdn.microsoft.com/commandline/2016/11/17/do-not-change-linux-files-using-windows-apps-and-tools/)

### 安装 zsh

* * *

> 目前常用的 Linux 系统和 OS X 系统的默认 Shell 都是 bash，但是真正强大的 Shell 是深藏不露的 zsh， 这货绝对是马车中的跑车，跑车中的飞行车，史称『终极 Shell』，但是由于配置过于复杂，所以初期无人问津，很多人跑过来看看 zsh 的配置指南，什么都不说转身就走了。直到有一天，国外有个穷极无聊的程序员开发出了一个能够让你快速上手的zsh项目，叫做「oh my zsh」，Github 网址是：[https://github.com/robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)。这玩意就像「X天叫你学会 C++」系列，可以让你神功速成，而且是真的。

zsh就是一款强大的虚拟终端，网上也都推荐使用 `oh my zsh` 来管理配置 , 不过对我来说还是不够傻瓜。于是，参考一篇[文章](https://www.zhihu.com/question/21418449/answer/300879747)使用 zsh 的包管理器：[antigen](https://github.com/zsh-users/antigen) 来管理所有功能，文章中还给了现成的配置。

*   安装 zsh

    sudo apt-get -y install zsh
    

*   设置终端的 shell 环境默认为 zsh，输入以下命令（需要重启）

    # 加 sudo 是修改 root 帐号的默认 shell
    chsh -s `which zsh`
    

*   如果上面命令无效，修改 `~/.bashrc` 在开头输入：

    if [ -t 1 ]; then
        exec zsh
    fi
    

*   安装 antigen

    curl -L git.io/antigen > antigen.zsh
    
    # 修改配置 ~/.zshrc（如果切换帐号后无法使用 zsh 则把该用户的配置文件再配一遍）
    curl -L https://raw.githubusercontent.com/skywind3000/vim/master/etc/zshrc.zsh > ~/.zshrc
    
    # 修改主题, 参考：https://github.com/robbyrussell/oh-my-zsh/wiki/themes
    # 如果需要主题一直生效需要添加到 ~/.zshrc 中
    antigen theme ys
    
    # 配置修改完重新执行 zsh
    

*   如果出现警告：**zsh compinit: insecure directories, run compaudit for list.**

    chmod -R 755 ~/.antigen
    

### 安装 autojump ([用法参考](https://linux.cn/article-3401-1.html))

* * *

> autojump 是一个命令行工具，它允许你可以直接跳转到你喜爱的目录，而不受当前所在目录的限制。意思就是可以让你更快地切换目录，而不用频繁地使用 cd/tab 等命令。

*   安装

    sudo apt-get install autojump
    

*   zsh 下运行报错

    $ autojump
    Please source the correct autojump file in your shell's
    startup file. For more information, please reinstall autojump
    and read the post installation instructions.
    

参照文章 [Mac终端增强技能](https://www.jianshu.com/p/0d4d5c0c31a1) 和 [终极 Shell](http://macshuo.com/?p=676) 找到解决办法：

> 在 `~/.zshrc` 中安装插件 `brew install autojump` 再重新进入 zsh

由于本文使用 antigen 作为 zsh 的包管理器，所以实际操作是在 `~/.zshrc` 中添加 `antigen bundle autojump`

### 使用 bash 客户端软件 cmder ([参考](http://blog.csdn.net/u010053050/article/details/52388663))

* * *

Windows 自带的命令提示符 cmd 并不是很好用（文本选中、复制粘贴等等操作），在这里使用 cmder 作为替代品，体验效果很好。去 [cmder 官网](http://cmder.net/)下载 [mini版](https://github.com/cmderdev/cmder/releases/download/v1.3.2/cmder_mini.zip)（完整版附带了模拟的bash环境，由于已经安装 linux 子系统，就不再需要了）的解压即可使用。

*   ##### 设置启动 **cmder** 时直接运行 bash:
    

    1. 进入 "Settings > Startup",
    2. 选择 "Startup options > Command line"，输入 "bash -cur_console:p"
    

pic5

启动直接运行bash

*   ##### 通过软件底部的加号按钮新开标签页并进入 bash
    

    1. 进入 "Settings > Startup > Tasks",
    2. 选择 "bash::bash"，将指令修改为 'cmd /c "bash" -cur_console:p -new_console:d:%USERPROFILE%'
    

`文中给的 cmder 是 1.3.2 版本的，最新的 1.3.6 版本已经默认集成了 WSL 的 Task，就不用这一步的配置了`

pic6

新标签页

*   ##### 设置配色主题
    

    1. 进入 "Settings > Features > Colors",
    2. "Schemes" 项下拉选择 "<ubuntu>"
    

`小贴士：将 cmder 目录添加到环境变量 path 中或者复制快捷方式到 "C:\Windows\System32" 目录下，就可通过 win + R 快捷键快速打开了`

### 运行图形界面程序([参考](http://blog.csdn.net/syrchina/article/details/52170440))

* * *

什么！linux 不是就这么个黑白界面的窗口吗？是的，你没看错，就是图形界面，这里用到的是 Xserver 这个东东，至于原理什么的各位自行度娘吧。

pic7

  

Xming

> 1.  安装Xming [下载地址](https://sourceforge.net/projects/xming/files/latest/download11)
>     
> 2.  安装完直接打开 `Xming` 即可
>     
> 3.  安装一个 `firefox` 测试  
>     `apt-get install firefox`
>     
> 4.  运行(在程序指令前加上"DISPLAY=:0 ")  
>     `DISPLAY=:0 firefox`
>     
> 5.  [简化配置](http://www.cnblogs.com/carlsplace/p/5990568.html)  
>     每次运行程序都要输入 `DISPLAY=:0` 挺累的，执行下列指令后重启bash即可省去这个步骤  
>     `echo "export DISPLAY=:0.0" >> ~/.bashrc`
>     
> 
> PS：很多小伙伴反映说打开Xming没反应，这是正常现象 ( 详见[评论区33楼](https://www.jianshu.com/p/bc38ed12da1d#comment-20826096) )。Xming是一个在后台运行的服务，在任务栏显示一个 `X` 的小图标即表示启动成功，无需其他操作了。要想看到画面，需要在wsl或其他远程Linux机器上使用 `DISPLAY=:0` 命令启动带图形界面的程序。在这里简单分析下我理解的原理：1. Xming 启动 Xserver服务用于监听；2. wsl启动程序后把界面数据发送给 Xserver；3. Xserver 接收到数据进行绘制，于是在win下就看到了图形界面。还有困惑的话请移步至下方的 vnc 版块，比Xming效果要好，类似与 win 下 `远程桌面连接` 的效果。

### Sublime Text 3 安装

* * *

既然都可以运行图形界面了，编辑器也要换成可视化的，毕竟vim还是不太适合我。

    cd /
    # 下载
    wget https://download.sublimetext.com/sublime-text_build-3126_amd64.deb
    # 安装
    dpkg -i sublime-text_build-3126_amd64.deb
    # 运行
    subl
    

pic8

  

什么鬼，报错了！

应该是少了什么依赖包，嗯，安装下搞定。

    apt-get -y install libgtk2.0-0
    

### 启动 xfce 桌面环境([参考](http://jingyan.baidu.com/article/546ae1851c6ac81149f28c9a.html))

* * *

图形界面程序都能运行了，不试试ubuntu的桌面环境怎么能甘心，于是又是噼里啪啦一顿搜索。一开始参照这篇[国外的教程](http://winaero.com/blog/run-ubuntu-unity-on-windows-10/)折腾了许久，每次运行总是报一个composite的插件没加载进来，各种软件包安装一通还是不行，后来实在失去耐心就放弃了这条路。后来看到好像有人成功运行了xfce,但没有具体步骤，只能自己一顿摸索，结果还真误打误撞成功了。

    # 1.安装xfce4
    apt-get install xfce4
    # 2.安装xubuntu桌面及附带应用
    apt-get install xubuntu-desktop
    # 3.启动
    xfce4-session
    

pic9

  

启动报错了

解决办法：（[参考](https://zhuanlan.zhihu.com/p/21577512?refer=MSFaith)）

    sed -i 's$<listen>.*</listen>$<listen>tcp:host=localhost,port=0</listen>$' /etc/dbus-1/session.conf
    

再次尝试打开，现在可以看到Xming打开了三个窗口，分别是桌面、任务栏、菜单栏。逼格是提升了不少，不过确实很卡。

pic10

  

xfce4

### 使用 [VNC](https://www.realvnc.com/en/connect/download/vnc/linux/) 进行远程桌面控制([安装方法](https://jingyan.baidu.com/article/d2b1d102b85a825c7e37d405.html))

* * *

> 感谢 [@lizr\_4bf0](https://www.jianshu.com/p/bc38ed12da1d#comment-20600481) 的提示，使用 `VNC` 来代替 `Xming` 以解决 `Xming` 下很卡的问题。

##### 1\. wsl 下安装 [vnc4server](https://www.linuxidc.com/Linux/2017-03/141936.htm)

    apt-get install vnc4server
    

##### 2\. wsl 下启动 `vncserver` ( 安装后首次启动需要设置访问密码 )

    vncserver
    

##### 3\. 在 win10 的 `VNC Viewer` 中访问 `127.0.0.1:1`

注：如果 `VNC Viewer` 连接报错请[参考](http://www.cnblogs.com/kerrycode/p/6055126.html)

##### 4\. `VNC Viewer` 中只显示一个终端窗口的问题

    # 修改xstartup, 将 x-window-manager 替换为刚才安装的 xfce4-session
    sed -i 's$x-window-manager$xfce4-session$' ~/.vnc/xstartup
    # 重启 vncserver
    vncserver -kill :1
    vncserver :1
    

##### 5\. [分辨率设置](http://blog.csdn.net/yingyujianmo/article/details/45201097)

    # 先关闭
    vncserver -kill :1
    # 再启动并设置分辨率(注意是小写的x)
    vncserver -geometry 1366x768 :1
    

### 在子系统上运行nginx

* * *

因工作项目中用到了 `ssi` 技术，而已经windows上已经编译好的 `nginx` 是不支持相对路径引用的（[需要修改源码重新编译](http://www.cnblogs.com/jyb2014/p/4026654.html)），只能委屈求全用着 `Apache` 。不过既然现在都能跑linux了，那就试试在linux上运行 `nginx`，然后在windows上进行调用。

##### 1\. 通过apt-get方式安装

    sudo apt-get install nginx
    # 启动报错了:
    nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
    # 80端口实际没被占用，那应该就是ipv6的问题，将其禁用: 
    vim /etc/nginx/sites-available/default
    # 找到default_server ipv6only=on;注释掉
    # 再次启动没报错，不过浏览器无法访问，80端口也没被使用，查看error.log
    cat /var/log/nginx/error.log
    # 看到错误信息：
    ioctl(FIOASYNC) failed while spawning "worker process" (22: Invalid argument)
    

    # 解决方法：禁用master进程模式
    sed -i '1 a\master_process off;' /etc/nginx/nginx.conf
    

再次启动，终于没报错了，Windows中打开浏览器访问127.0.0.1，还真的实验成功了，不过nginx版本有点老，是1.4.6的。

##### 2\. 通过编译源码的方式安装

    # 1.安装依赖包
    apt-get -y install build-essential autoconf libtool libxml2-dev openssl libcurl4-openssl-dev libbz2-dev libjpeg-dev libpng12-dev libfreetype6-dev libldap2-dev libmcrypt-dev libmysqlclient-dev libxslt1-dev libxt-dev libpcre3-dev libreadline-dev
    # 2.下载源码
    wget http://tengine.taobao.org/download/tengine-2.1.1.tar.gz
    # 3.解压
    tar -zxvf tengine-2.1.1.tar.gz
    # 4.进入目录
    cd tengine-2.1.1
    # 修改源码...
    # 5.配置
    ./configure --prefix=/usr/anyesu/nginx
    # 6.编译&安装
    make && make install
    # 7.修改配置文件
    sed -i '1 a\master_process off;' /usr/anyesu/nginx/conf/nginx.conf
    # 8.启动
    /usr/anyesu/sbin/nginx
    

上面的步骤，我试了两台电脑，其中一台报错：

> _nginx: \[emerg\] invalid port in resolver "fec0:0:0:ffff::1" in /usr/anyesu/nginx/conf/nginx.conf:123_

pic11

  

/etc/resolv.conf

出现的 `fec0:0:0:ffff::1` 是个什么鬼，度娘了一番，貌似是dsn，打开dns配置文件 `/etc/resolv.conf` 果然发现了这东西，应该是Windows下只分配了1个dns，所以linux就给了这么两个默认值的吧。将它们注释掉，重新启动nginx，成功运行 [Tengine/2.1.1](http://tengine.taobao.org)

`注意，每次重启bash都会重置dns配置的`

### 启用ssh([参考](http://blog.csdn.net/donglynn/article/details/53505495))

* * *

本地可以通过命令行打开bash，如果要远程访问（如同访问线上服务器一样），那么就需要启用ssh。

    # 1.安装ssh(一般不需要这步)
    apt-get install openssh-server
    # 2.修改配置文件
    cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
    vim /etc/ssh/sshd_config
    #=======(修改以下选项内容)=========#
    # Port 23（22端口已被占用）        #
    # (取消注释)ListenAddress 0.0.0.0 #
    # UsePrivilegeSeparation no      #
    # PermitRootLogin yes            #
    # （注释）StrictModes yes         #
    # PasswordAuthentication yes     #
    #================================#
    # 3.启动ssh
    service ssh start
    # 4.如果提示“sshd error: could not load host key”，则用下面的命令重新生成
    rm /etc/ssh/ssh*key
    dpkg-reconfigure openssh-server
    

使用终端工具访问，这里我用的是 `WinSCP` + `PuTTY`

pic12

WinSCP

pic13

  

PuTTY

### WSL开机启动

* * *

pic14

  

任务管理器

打开任务管理器我们可以发现，在运行子系统的时候，后台其实有一个bash的进程在运行，我们每开一个命令行窗口就会多一个 `bash.exe`，如果我们把所有的 `bash.exe` 都关闭则这个bash进程就关闭了（相当于是子系统关机了），跟着之前在子系统中打开的程序如nginx、sshd等也随之关闭了。为了让linux程序能够在后台继续运行，这里通过 `vbs` 脚本在后台打开一个 `bash.exe` 来保证bash进程一直开着。当然，还可以通过Windows的计划任务实现开机启动WSL并打开其中的程序。  
`注：目前1803版本中关闭bash.exe不会再关闭打开的linux进程了，也就是说不需要再在后台维持一个bash.exe`

    # 1.新建开机启动脚本
    vim /usr/anyesu/boot.sh
    # 2.编辑
    #================================================================
    # #!/bin/sh
    # /usr/anyesu/nginx/sbin/nginx
    # service ssh start
    # $SHELL  #这句很重要，挂起当前脚本进程,进而维持base.exe一直打开
    #================================================================
    # 3.设置权限
    chmod 777 /usr/anyesu/boot.sh
    # 4.创建vbs脚本(D:\linux\wsl.vbs)
    #==========================================
    # Set ws = CreateObject("Wscript.Shell") 
    # ws.run "bash /usr/anyesu/boot.sh",vbhide
    #==========================================
    # 5.创建计划任务
    

pic15

计划任务

pic16

创建基本任务

pic17

  

设置任务名称

pic18

  

设置任务触发条件——用户登录

pic19

  

设置任务操作——启动

pic20

  

设置任务操作——运行vbs脚本

pic21

运行计划任务

### 使 WSL 支持 32位程序

* * *

*   参考 [issue#2468](https://github.com/Microsoft/WSL/issues/2468#issuecomment-374904520) ([中文](https://github.com/Microsoft/WSL/issues/2468#issuecomment-435650594))  
    `注: 依赖包可能无法自动安装, 自己手动安装下`

### 关于Linux发行版本

* * *

*   #### [升级Ubuntu版本至Xenial(16.04)](http://www.cnblogs.com/leehavin/p/5751762.html)
    
*   #### [使用openSUSE替换Ubuntu](http://115.com/1000079591/T998027.html)
    

    # 打开cmd，进入bash
    bash
    cd /
    # 下载openSUSE
    wget -O openSUSE-42.2.tar.xz https://github.com/openSUSE/docker-containers-build/blob/openSUSE-42.2/docker/openSUSE-42.2.tar.xz?raw=true
    # 解压
    xz -d /openSUSE-42.2.tar.xz
    mkdir rootfs
    tar -C rootfs -xvf openSUSE-42.2.tar
    # 退出bash,返回cmd
    exit
    # 进入子系统所在路径
    cd %localappdata%\lxss
    # 备份ubuntu用户空间（看下任务管理器中bash是不是彻底关闭了）
    rename rootfs rootfs.ubuntu
    # 使用openSUSE用户空间替换默认用户空间
    move rootfs.ubuntu\rootfs rootfs
    # 设置默认登录用户
    lxrun /setdefaultuser root
    # 重新进入bash
    bash
    #查看发行版本
    cat /etc/issue
    

### 官方资料

* * *

*   #### [github](https://github.com/Microsoft/BashOnWindows)
    
*   #### [博客](https://blogs.msdn.microsoft.com/commandline/)
    
*   #### [insider build的更新记录](https://msdn.microsoft.com/en-us/commandline/wsl/release_notes)
    

### 2017-11-21追加

* * *

最近系统又被强更到了1709(16299.64)，发现几点变化做个记录:

1.  ping命令已经可以正常使用了
2.  nginx的master模式也能正常使用不会报错了
3.  发现nginx、ssh之类的，能正常启动不报错但怎么也无法绑定端口。后来查了github上的 [issues](https://github.com/Microsoft/WSL/issues/1554#issuecomment-341407417) 发现是wegame(原tgp)的锅，原因是使用了一个win10上已失效的特性，wegame的开发表示会尽快修复。临时解决办法：删除文件 `%systemroot%\system32\drivers\QMTgpNetflow764.sys` 后重启bash，如无法删除先关闭应用或卸载再重装wegame(最好重启电脑)，重装后先删除 `QMTgpNetflow764.sys` 再运行wegame。
4.  内核升级为 `4.4.0-43-Microsoft` 了, 带上了微软的标记，推测是这个原因导致很多软件包无法正常安装了。后来发现应该是阿里云的数据源未同步的原因，加上 **ubuntu** 自带的源 ( 即文中最初备份的内容 ) 即可解决。

*   [Windows 10 Fall Creators Update (1709) 中WSL的新功能](https://blogs.msdn.microsoft.com/commandline/2017/10/11/whats-new-in-wsl-in-windows-10-fall-creators-update/)

### 2017-11-30追加

* * *

pic22

  

应用商店

目前 ( **1709** 版本 `16299.64` ) 已经可以在商店中搜索安装多个不同版本的子系统了，根路径为 `%localappdata%\Packages\【根据子系统名找到对应的应用文件夹】\LocalState\rootfs` 。同时还新增了两个命令行工具: `wsl.exe` 和 `wslconfig.exe` 。

其中 `wsl.exe` 应该等价于 `bash.exe` , 两者之间的细微差别暂时还没发现。至于 `wslconfig.exe` 的作用主要为([参考](https://blogs.msdn.microsoft.com/commandline/2017/11/28/a-guide-to-invoking-wsl))：

> **1\. 查看安装所有已安装的子系统: `wslconfig /l`**
> 
>     适用于 Linux 的 Windows 子系统:
>     Ubuntu (默认)
>     Legacy
>     
> 
> 其中 `Ubuntu` 是商店中下载的版本，`Legacy` 是按老方法安装的默认wsl。
> 
> **2\. 切换bash.exe默认使用的子系统: `wslconfig /s <DistributionName>`**  
> 其中 `<DistributionName>` 替换为 `Ubuntu` 或 `Legacy` , 或者其他已安装的子系统。
> 
> **3\. 卸载已安装的子系统: `wslconfig /u <DistributionName>`**  
> 同上替换 `<DistributionName>` 。经测试发现，此 `"卸载"` 并不会卸载商店中安装的 `Ubuntu` 应用, 即再次执行该应用又会重新安装了。

### 2018-10-11追加

* * *

官方博客中给出了命令行方式来安装指定版本的 **WSL** ( [参考](https://blogs.msdn.microsoft.com/commandline/2018/05/15/build-2018-recap/) )  
以管理员权限启动 **PowerShell** ( 快捷键 `WIN + X` 调出 ) 执行下面命令

*   启用 **WSL** 特性

    # 会提示重启电脑
    Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
    

*   安装 **Ubuntu**

    # 下载安装包
    Invoke-WebRequest -Uri https://aka.ms/wsl-ubuntu-1604 -OutFile ~/Ubuntu.appx -UseBasicParsing
    # 安装
    Add-AppxPackage -Path ~/Ubuntu.appx
    # 启动
    Ubuntu.exe
    

*   安装 **SLES**

    # 下载安装包
    Invoke-WebRequest -Uri https://aka.ms/wsl-sles-12 -OutFile ~/SLES.appx -UseBasicParsing
    # 安装
    Add-AppxPackage -Path ~/SLES.appx
    # 启动
    sles-12.exe
    

*   安装 **openSUSE**

    # 下载安装包
    Invoke-WebRequest -Uri https://aka.ms/wsl-opensuse-42 -OutFile ~/openSUSE.appx -UseBasicParsing
    # 安装
    Add-AppxPackage -Path ~/openSUSE.appx
    # 启动
    opensuse-42.exe
    

### 写在最后

* * *

WSL折腾完有一段时间了，只是一直没时间记录下来（也许是懒吧）。在此之前，由于工作需要，偶尔兼职运维的角色，折腾下服务器什么的，就很业余的学习了一些linux指令。以前装过vmware，体验不是很好就不想装了，所以写shell脚本、编译源码什么的都是在公司测试服务器上练习的，现在有了WSL之后就可以在自己本地练习了(肆意折腾，哈哈哈)。使用方面，体验和使用终端工具连接远程服务器是差不多的；性能方面，子系统(bash进程)本身是不占多少内存的，启动程序几乎相当于启动Windows程序了，不显示图形界面内存都占用比较小，肯定优于"印象中的虚拟机"。总的来说，WSL还是比较值得推荐去折腾的，也比较适合新手学习linux，虽然我也只是个小白








