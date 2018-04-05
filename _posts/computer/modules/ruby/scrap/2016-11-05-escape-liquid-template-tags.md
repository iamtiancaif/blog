---
layout: post
title:  "escape liquid template tags"
categories: computer
tags:  computer jekyll copy stackoverflow
author: web
source: https://stackoverflow.com/questions/3426182/how-to-escape-liquid-template-tags/5866429
---

* content
{:toc}



earchers, there _is_ a way to escape without plugins, use the code below:
{% raw %}
    {{ "{% this " }}%}

and for tags, to escape `{{ this }}` use:

    {{ "{{ this " }}}}
{% endraw %}    
    

* * *


There is also a jekyll plugin for this which makes it a whole lot easier: [https://gist.github.com/1020852](https://gist.github.com/1020852)
> Raw tag for jekyll. Keeps liquid from parsing text betweeen {{ "{% raw " }}%} and {{ "{% endraw " }}%}
