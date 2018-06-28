---
layout: post
title:  "SpringBoot使用Swagger2出现Unable to infer base url"
categories: computer
tags:  computer copy java
author: "Seayon阿阳"
source: "https://zhaoxuyang.com/springboot%E4%BD%BF%E7%94%A8swagger2%E5%87%BA%E7%8E%B0unable-to-infer-base-url-this-is-common-when-using-dynamic-servlet-registration-or-when-the-api-is-behind-an-api-gateway/"
---

* content
{:toc}


在使用SpringBoot中配置Swagger2的时候，出现

Unable to infer base url. This is common when using dynamic servlet registration or when the API is behind an API Gateway. The base url is the root of where all the swagger resources are served. For e.g. if the api is available at http://example.org/api/v2/api-docs then the base url is http://example.org/api/. Please enter the location manually:

是要在SpringBoot的启动Application前面加上 @EnableSwagger2注解
















