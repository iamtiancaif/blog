---
layout: post
title:  "convert HashMap to JsonNode with Jackson"
categories: computer
tags:  computer copy stackoverflow java mybatis
author: "web"
source: "https://stackoverflow.com/questions/26704101/mybatis-mapper-file-escape-characters/26728966"
---

* content
{:toc}


You can use `CDATA` to escape special characters.

    <select id="selectMonday" resultType="SheetGameRec">
        select ColumnName
        from Table
        where ColumnName <![CDATA[ <= 2 ]]>
        order by ColumnName;
    </select>

Another option is encoded chars. eg: `colName &lt; 2 OR colName &gt; 10`




