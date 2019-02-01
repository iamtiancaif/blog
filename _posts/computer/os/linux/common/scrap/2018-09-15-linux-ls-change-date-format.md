---
layout: post
title:  "linux ls change date format"
categories: computer
tags:  computer copy stackoverflow linux
author: "web"
source: "https://unix.stackexchange.com/questions/413788/how-to-convert-file-dates-to-a-particular-format-when-using-ls"
---

* content
{:toc}


If you're using GNU `ls` (this is standard on Linux systems), it has a `--time-style` option which can be used to change the date/time format.

The built-in format closest to what you want is `long-iso`.

e.g.

    $ ls -l --time-style=long-iso
    total 1
    -rw-r--r-- 1 cas cas 0 2017-12-30 21:26 ffd_ik_imp_bus.dat
    -rw-r--r-- 1 cas cas 0 2017-12-30 21:26 ffd_in_imp_bus.dat

You can also use a custom format using the same date formatting specification as in GNU `date`:

    $ ls -l --time-style='+%Y%m%d %H:%M'
    total 1
    -rw-r--r-- 1 cas cas 0 20171230 21:26 ffd_ik_imp_bus.dat
    -rw-r--r-- 1 cas cas 0 20171230 21:26 ffd_in_imp_bus.dat

From `man ls`:

> **--time-style=STYLE**
> 
> with `-l`, show times using style STYLE: `full-iso`, `long-iso`, `iso`, `locale`, or `+FORMAT`
> 
> FORMAT is interpreted like in `date'
> 
> if FORMAT is `FORMAT1<newline>FORMAT2`, then FORMAT1 applies to non-recent files and FORMAT2 to recent files.
> 
> if STYLE is prefixed with `posix-`, STYLE takes effect only outside the POSIX locale




