---
layout: post
title: "url decoding in java"
categories: computer
tags: computer java scrap stackoverflow
author: web
source: 'https://stackoverflow.com/questions/6138127/how-to-do-url-decoding-in-java'
---

* content
{:toc}

This does not have anything to do with character encodings such as UTF-8 or ASCII. The string you have there is _URL encoded_. This kind of encoding is something entirely different than character encoding.

Try something like this:

    String result = java.net.URLDecoder.decode(url, "UTF-8");

Note that a _character encoding_ (such as UTF-8 or ASCII) is what determines the mapping of characters to raw bytes. For a good intro to character encodings, see [this article](http://www.joelonsoftware.com/articles/Unicode.html).


