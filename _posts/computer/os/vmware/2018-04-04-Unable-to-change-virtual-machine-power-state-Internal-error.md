---
layout: post
title:  "Unable to change virtual machine power state Internal error"
categories: computer
tags:  computer vmware  copy
author: web 
source: https://www.surlyjake.com/blog/2009/08/14/vmware-unable-to-change-virtual-machine-power-state-internal-error/
---

* content
{:toc}


Ran into this while running Vmware Workstation under Ubuntu Jaunty. I got an error while shutting down the machine through an NX session.

This is a result of a zombie 'vmware-vmx' process running. All you need to do is kill the process. This command sends 'signal 9' to the process. When sent to a program, SIGKILL causes it to terminate immediately. In contrast to SIGTERM and SIGINT, this signal cannot be caught or ignored. For more information: [more sigkill info](http://en.wikipedia.org/wiki/SIGKILL).

	killall -s9 vmware-vmx

After that, I was able to start up the virtual machine without issue.
