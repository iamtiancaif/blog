---
layout: post
title:  "Convert JsonNode object to Map"
categories: computer
tags:  computer copy stackoverflow java json
author: "web"
source: "https://stackoverflow.com/questions/26766256/convert-jsonnode-object-to-map"
---

* content
{:toc}


Basically just use the `ObjectMapper` to convert the value for you:

    ObjectMapper mapper = new ObjectMapper();
    Map<String, Object> result = mapper.convertValue(jsonNode, Map.class);














