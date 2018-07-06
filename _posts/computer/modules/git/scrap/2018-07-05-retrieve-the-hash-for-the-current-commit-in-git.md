---
layout: post
title:  "retrieve the hash for the current commit in git"
categories: computer
tags:  computer copy stackoverflow git
author: "web"
source: "https://stackoverflow.com/questions/949314/how-to-retrieve-the-hash-for-the-current-commit-in-git"
---

* content
{:toc}


To turn arbitrary extended object reference into SHA-1, use simply **[git-rev-parse](http://git-scm.com/docs/git-rev-parse "git-rev-parse - Pick out and massage parameters")**, for example

    git rev-parse HEAD

or

    git rev-parse --verify HEAD

**_Sidenote:_** If you want to turn _references_ (**branches** and **tags**) into SHA-1, there is `git show-ref`and `git for-each-ref`.

`--verify` implies that: `The parameter given must be usable as a single, valid object name. Otherwise barf and abort.`  

`git rev-parse --short HEAD` returns the short version of the hash, just in case anyone was wondering.  

`--short=N` is about **minimal** number of digits; git uses larger number of digits if shortened one would be undistinguishable from shortened other commit. Try e.g. `git rev-parse --short=2 HEAD` or `git log --oneline --abbrev=2`  




