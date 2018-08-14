---
layout: post
title:  "pad an integer with zeros on the left"
categories: computer
tags:  computer copy java stackoverflow
author: "web"
source: "https://stackoverflow.com/questions/473282/how-can-i-pad-an-integer-with-zeros-on-the-left"
---

* content
{:toc}


Use java.lang.String.format(String,Object...) like this:

	String.format("%05d", yournumber);

for zero-padding with a length of 5. For hexadecimal output replace the `d` with an `x` as in `"%05x"`.

The full formatting options are documented as part of `java.util.Formatter`.


