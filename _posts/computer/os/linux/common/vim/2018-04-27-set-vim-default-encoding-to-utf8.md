---
layout: post
title:  "set vim default encoding to utf8"
categories: computer
tags:  computer linux vim stackexchange
author: web
source: "https://unix.stackexchange.com/questions/23389/how-can-i-set-vims-default-encoding-to-utf-8"
---

* content
{:toc}


When Vim reads an existing file, it tries to detect the file encoding. When writing out the file, Vim uses the file encoding that it detected (except when you tell it differently). So a file detected as UTF-8 is written as UTF-8, a file detected as Latin-1 is written as Latin-1, and so on.

By default, the detection process is crude. Every file that you open with Vim will be assumed to be Latin-1, unless it detects a Unicode byte-order mark at the top. A UTF-8 file without a byte-order mark will be hard to edit because any multibyte characters will be shown in the buffer as character sequences instead of single characters.

Worse, Vim, by default, uses Latin-1 to represent the text in the buffer. So a UTF-8 file _with_ a byte-order mark will be corrupted by down-conversion to Latin-1.

The solution is to configure Vim to use UTF-8 internally. This is, in fact, recommended in the Vim documentation, and the only reason it is not configured that way out of the box is to avoid creating enormous confusion among users who expect Vim to operate basically as a Latin-1 editor.

In your `.vimrc`, add `set encoding=utf-8` and restart Vim.

> Or instead, set the `LANG` environment variable to indicate that UTF-8 is your preferred character encoding. This will affect not just Vim but any software which relies on `LANG` to determine how it should represent text. For example, to indicate that text should appear in English (`en`), as spoken in the United States (`US`), encoded as UTF-8 (`utf-8`), set `LANG=en_US.utf-8`.

Now Vim will use UTF-8 to represent the text in the buffer. Plus, it will also make a more determined effort to detect the UTF-8 encoding in a file. Besides looking for a byte-order mark, it will also check for UTF-8 without a byte-order mark before falling back to Latin-1. So it will no longer corrupt a file coded in UTF-8, and it should properly display the UTF-8 characters during the editing session.

For more information on how Vim detects the file encoding, see [the `fileencodings` option in the Vim documentation](http://vimdoc.sourceforge.net/htmldoc/options.html#%27fileencodings%27).

For more information on setting the encoding that Vim uses internally, see [the `encoding` option](http://vimdoc.sourceforge.net/htmldoc/options.html#%27encoding%27).

If you need to override the encoding used when writing a file back to disk, see [the `fileencoding` option](http://vimdoc.sourceforge.net/htmldoc/options.html#%27fileencoding%27).




something still in doubt
------------------------

**jxz**:  
In your `.vimrc`, add `set fileencoding=utf-8`.


>fileencodings=utf-8 will cause Vim to recognize the input file as UTF-8 but then perform a lossy conversion to Latin-1. Plus it will cause Vim to fail to recognize UTF-16. The better solution is to set encoding=utf-8 which turns Vim from a native one-byte editor into a native multibyte editor.



