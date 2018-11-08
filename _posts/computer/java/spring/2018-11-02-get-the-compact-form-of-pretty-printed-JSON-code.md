---
layout: post
title: "get the compact form of pretty printed JSON code"
categories: computer
tags: computer java stackoverflow
author: web
source: 'https://stackoverflow.com/questions/12173416/how-do-i-get-the-compact-form-of-pretty-printed-json-code'
---

* content
{:toc}


Jackson allows you to read from a JSON string, so read the pretty-printed string back into Jackson and then output it again with pretty-print disabled.

See [converting a String to JSON](https://stackoverflow.com/questions/6591388/converting-a-string-to-json-in-java)

**Simple Example**

        String prettyJsonString = "{ \"Hello\" : \"world\"}";
        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode jsonNode = objectMapper.readValue(prettyJsonString, JsonNode.class);
        System.out.println(jsonNode.toString());

**Requires**

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.5.3</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.5.3</version>
    </dependency>


