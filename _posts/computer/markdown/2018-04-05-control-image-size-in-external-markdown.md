---
layout: post
title:  "control image size in external markdown"
categories: computer
tags:  computer copy markdown jekyll
author: web
source: https://github.com/hakimel/reveal.js/issues/1349
---

* content
{:toc}


if you are using kramdown, you can do this:

    ![test image size](/img/post-bg-2015.jpg){:class="img-responsive"}
    ![test image size](/img/post-bg-2015.jpg){:height="50%" width="50%"}
    ![test image size](/img/post-bg-2015.jpg){:height="700px" width="400px"}



