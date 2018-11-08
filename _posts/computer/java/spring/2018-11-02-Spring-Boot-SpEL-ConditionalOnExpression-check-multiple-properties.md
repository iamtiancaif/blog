---
layout: post
title: "Spring Boot SpEL ConditionalOnExpression check multiple properties"
categories: computer
tags: computer java spring scrap
author: web
source: 'https://stackoverflow.com/questions/40477251/spring-boot-spel-conditionalonexpression-check-multiple-properties'
---

* content
{:toc}


The annotations `@ConditionalOnProperty` and `@ConditionalOnExpression` both do NOT have the `java.lang.annotation.Repeatable` annotation so you would not be able to just add multiple annotations for checking multiple properties.

The following syntax has been tested and works:

**Solution for Two Properties**

    @ConditionalOnExpression("${properties.first.property.enable:true} && ${properties.second.property.startServer:false}")

Note the following:

*   You need to using colon notation to indicate the default value of the property in the expression language statement
*   Each property is in a separate expression language block ${}
*   The && operator is used outside of the SpEL blocks

It allows for multiple properties that have differing values and can extend to multiple properties.

If you want to check more then 2 values and still maintain readability, you can use the concatenation operator between different conditions you are evaluating:

**Solution for more then 2 properties**

    @ConditionalOnExpression("${properties.first.property.enable:true} " +
            "&& ${properties.second.property.enable:true} " +
            "&& ${properties.third.property.enable:true}")

The drawback is that you cannot use a matchIfMissing argument as you would be able to when using the `@ConditionalOnProperty` annotation so you will have to ensure that the properties are present in the _.properties_ or _YAML_ files for all your profiles/environments or just rely on the default value



