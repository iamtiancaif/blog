---
layout: post
title:  "NoClassDefFoundError: org.apache.http.ssl.SSLContexts"
categories: computer
tags:  computer copy java
author: "web"
source: "https://blog.csdn.net/HeatDeath/article/details/79411540"
---

* content
{:toc}


添加依赖接口解决

            <dependency>
                <groupId>org.apache.httpcomponents</groupId>
                <artifactId>httpcore</artifactId>
                <version>4.4</version>
            </dependency>

            <dependency>
                <groupId>org.apache.httpcomponents</groupId>
                <artifactId>httpclient</artifactId>
                <version>4.5.2</version>
            </dependency>



