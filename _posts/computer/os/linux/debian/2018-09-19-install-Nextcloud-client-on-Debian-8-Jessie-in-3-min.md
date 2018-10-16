---
layout: post
title: "install Nextcloud client on Debian 8 Jessie in 3 min"
categories: computer
tags:  computer linux debian scrap
author: web
source: "https://www.jurisic.org/index.php?post/2017/02/12/How-to-install-Nextcloud-client-on-Debian-8-Jessie-in-3-min"
ignore: true
---

* content
{:toc}


Nextcloud client is forked from ownCloud client and based on version 2.2.4, Built from Git revision [eaeed0](https://github.com/owncloud/client/commit/eaeed08544d1d7f4031d28a8e1bd9dd5e08a60fd) on Feb 7 2017, 20:21:04 using Qt 5.3.2, OpenSSL 1.0.1t 3 May 2016.

Add Nextcloud Repository
------------------------

Add the Nextcloud repository in the /etc/apt/sources.d/nextcloud.list

echo 'deb http://apt.jurisic.org/debian/ jessie main contrib non-free' >> /etc/apt/sources.list.d/nextcloud.list

Install release key of Nextcloud repository:

wget -q http://apt.jurisic.org/Release.key -O- | apt-key add -

And run apt-get update to download the list of packages.

apt-get update

Install Nextcloud client
------------------------

Install Nextcloud server with:

apt-get install nextcloud-client

Choose "y" to start the installation.

After installation Nextcloud client is ready for use, simple click on Aplications->Accessories->**Nextcloud desktop sync** client and enjoy.



