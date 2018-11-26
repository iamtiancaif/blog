---
layout: post
title: "svn ignore specific file types in all sub-directories"
categories: computer
tags:  computer copy svn
author: web 
source: "https://stackoverflow.com/questions/6878673/svn-subversion-ignore-specific-file-types-in-all-sub-directories-configured-o"
---

* content
{:toc}


I believe the answer I was looking for is:

    vim ~/.subversion/config

scroll down to 'global-ignores':

    global-ignores = *.classpath

The answer came from here:

[How can I recursively configure svn status to hide ignored files?](https://stackoverflow.com/questions/1594933/how-can-i-recursively-configure-svn-status-to-hide-ignored-files)







