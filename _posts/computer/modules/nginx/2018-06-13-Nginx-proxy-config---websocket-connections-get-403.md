---
layout: post
title: "Nginx proxy config - websocket connections get 403"
categories: computer
tags:  computer nginx copy
author: "web"
source: "https://forum.mattermost.org/t/nginx-proxy-config-websocket-connections-get-403/2924"
---

* content
{:toc}


The reason is most likely a new cross-origin check: [https://gitlab.com/gitlab-org/omnibus-gitlab/issues/1966 105](https://gitlab.com/gitlab-org/omnibus-gitlab/issues/1966)  
Fixed it by adding headers to the nginx forwarding:

     location /api/v3/users/websocket {
      proxy_pass            http://127.0.0.1:8065;
      (...)
      proxy_set_header      Host (your host name);
      proxy_set_header      X-Forwarded-For $remote_addr;
    }











