---
layout: post
title:  "Jackson Remove double quotes"
categories: computer
tags:  computer copy stackoverflow java json
author: "web"
source: "https://stackoverflow.com/questions/28646572/faster-xml-jackson-remove-double-quotes"
---

* content
{:toc}


Question
--------

I have the following json:

    {"test":"example"}

I use the following code from Faster XML Jackson.

    JsonParser jp = factory.createParser("{\"test\":\"example\"}");
    json = mapper.readTree(jp);
    System.out.println(json.get("test").toString());

It outputs:

    "example"

Is there a setting in Jackson to remove the double quotes?

 

Answer
------

Well, what you obtain when you `.get("test")` is a `JsonNode` and it happens to be a `TextNode`; when you `.toString()` it, it will return the string representation of that `TextNode`, which is why you obtain that result.

What you want is to:

    .get("test").textValue();

which will return the actual content of the JSON String itself (with everything unescaped and so on).

Note that this will return null if the `JsonNode` _is not_ a `TextNode`.











