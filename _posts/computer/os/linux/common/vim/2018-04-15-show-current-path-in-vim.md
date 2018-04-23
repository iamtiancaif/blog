---
layout: post
title:  "show current path in vim"
categories: computer
tags:  computer os vim copy stackexchange
author: web 
ignore: true
source: https://vi.stackexchange.com/questions/104/how-can-i-see-the-full-path-of-the-current-file
---

* content
{:toc}


You can press `1` followed by `Ctrl+G` to see the full path of the current file.

(Pressing only `Ctrl+G` shows the path relative to Vim's current working directory, as pointed out by Jasper in the comments.)

You can use the following command in your `.vimrc` to add the full path to the status line, so it is always visible:

    set statusline+=%F



