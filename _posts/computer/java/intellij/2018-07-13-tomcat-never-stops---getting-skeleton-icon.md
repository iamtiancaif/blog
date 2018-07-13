---
layout: post
title:  "tomcat never stops - getting skeleton icon"
categories: computer
tags:  computer java intellij
author: "web"
source: "https://intellij-support.jetbrains.com/hc/en-us/community/posts/206213219-tomcat-never-stops-getting-skeleton-icon"
---

* content
{:toc}

When you press 'Stop' button IDEA just calls the shutdown script configured on 'Startup/Connection' tab of the Tomcat run configuration, which uses 'catalina.bat stop' command by default. So it looks like you have some problem in your application or the Tomcat instance which prevents Tomcat from stopping.


stack overflow
---------------

https://stackoverflow.com/questions/23553018/spring-boot-tomcat-termination  

Here is a maven example for spring boot actuator to configure HTTP endpoint to shutdown a spring boot web app:

1.Maven Pom.xml:

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

2.application.properties:

    #No auth  protected 
    endpoints.shutdown.sensitive=false
    
    #Enable shutdown endpoint
    endpoints.shutdown.enabled=true

All endpoints are listed [here](http://%20http//docs.spring.io/spring-boot/docs/1.3.3.RELEASE/reference/htmlsingle/#production-ready-customizing-endpoints):

3.Send a post method to shutdown the app:

    curl -X POST localhost:port/shutdown


