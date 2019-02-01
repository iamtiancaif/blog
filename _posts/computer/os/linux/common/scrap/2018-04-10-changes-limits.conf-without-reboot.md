---
layout: post
title:  "changes limits.conf without reboot"
categories: computer
tags:  computer copy stackexchange linux
author: "web"
source: "https://unix.stackexchange.com/questions/108603/do-changes-in-etc-security-limits-conf-require-a-reboot"
---

* content
{:toc}


These limits will be applied after reboot.

If you want to apply changes without reboot, modify  /etc/pam.d/common-session by adding this line at the end of file:

	session required pam_limits.so



