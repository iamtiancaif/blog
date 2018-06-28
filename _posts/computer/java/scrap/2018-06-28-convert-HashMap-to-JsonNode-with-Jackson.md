---
layout: post
title:  "convert HashMap to JsonNode with Jackson"
categories: computer
tags:  computer copy stackoverflow java json
author: "web"
source: "https://stackoverflow.com/questions/39391095/how-to-convert-hashmap-to-jsonnode-with-jackson"
---

* content
{:toc}


The following will do the trick:

    JsonNode jsonNode = mapper.convertValue(map, JsonNode.class);

Or use the _more elegant_ solution pointed in the [comments](https://stackoverflow.com/questions/39391095/how-to-convert-hashmap-to-jsonnode-with-jackson/39393002#comment75755795_39393002):

    JsonNode jsonNode = mapper.valueToTree(map);

* * *

If you need to write your `jsonNode` as a string, use:

    String json = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(jsonNode);








