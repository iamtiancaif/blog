---
layout: post
title:  "install Jekyll on Debian 8"
categories: computer
tags:  computer os ruby jekyll copy
author: web
source: https://www.rosehosting.com/blog/how-to-install-jekyll-on-debian-8/
---

* content
{:toc}

Jekyll is a static website generator written in Ruby. By using Jekyll you can create personal websites or websites for organizations without using databases. The main feature of Jekyll is that it can transform the plain text to a static website by rendering Markdown or Textile and Liquid templates. Jekyll is used as an engine behind GitHub Pages. Today we will show you how to install it on a [Linux VPS](https://www.rosehosting.com/).  

To install Jekyll you need to log in to your virtual server via SSH. Once you are logged in, you should update all your system software to the latest version available before you continue. Therefore, execute the following commands one by one:

	sudo apt-get update
	sudo apt-get upgrade

Next, you need to install Ruby on your[ Debian VPS](https://www.rosehosting.com/debian-vps.html). This is because Jekyll is written in Ruby and it is a major requirement. Go ahead an install Ruby by executing the following command:

	sudo apt-get install ruby-full build-essential 

This will take a few moments. After the process is completed, you can verify that Ruby is successfully installed by using the following command:

	ruby -v

You should get something like this:

ruby 2.1.5p273 (2014-11-13) \[x86_64-linux-gnu\]
<!--more-->

Jekyll 2 requires Ruby v1.9.3 or above and Jekyll 3 requires Ruby v2 or above. As you can see in the previous output, our version is 2.1.5p273, which means we are good to go.

To install Jekyll you need to execute the following command:

	gem install jekyll

This should take a few seconds. To verify that Jekyll is successfully installed and to find out the version, you can execute the command below:

	jekyll -v

The output of the command should be similar to this one:

jekyll 3.0.0


This means, Jekyll 3.0.0 is installed on our system.  
Now, lets create a test project using Jekyll and access the project using a web browser. Navigate to your ‘/home’ directory:

	cd home/

To create a Jekyll project, run the following command:

	jekyll new jekyll_project

Navigate in to your ‘jekyll_project’ directory:

	cd jekyll_project/

To start your first Jekyll application you need to execute the following command:

	jekyll serve --host 111.111.111.111 &

Please note, you need to replace ‘111.111.111.111’ with the actual IP address of your[ Debian VPS](https://www.rosehosting.com/debian-vps.html). By default your Jekyll application should listen on port 4000, so to access your application open your favorite web browser and navigate to:

http://111.111.111.111:4000

Again, do not forget to replace the IP address we used as an example with the actual IP address of your server.

To learn how to get started with Jekyll, we recommend you to read the official Jekyll documentation available at [https://jekyllrb.com/docs/home/](https://jekyllrb.com/docs/home/). There you can find many useful information about how to configure Jekyll, the basic usage, manage your content etc.

Of course you don’t have to do any of this if you use one of our [Linux VPS hosting](https://www.rosehosting.com/linux-vps-hosting.html) services, in which case you can simply ask our expert Linux admins to install Jekyll for you. They are available 24×7 and will take care of your request immediately.

PS. If you liked this post please share it with your friends on the social networks using the buttons on the left or simply leave a reply below. Thanks.





