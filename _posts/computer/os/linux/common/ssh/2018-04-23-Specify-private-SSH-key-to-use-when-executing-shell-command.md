---
layout: post
title:  "Specify private SSH-key to use when executing shell command"
categories: computer
tags:  computer linux ssh stackoverflow
author: web
source: https://stackoverflow.com/questions/4565700/specify-private-ssh-key-to-use-when-executing-shell-command
ignore: true
---

* content
{:toc}

nstead, I elaborate on @Martin v. Löwis 's mention of setting a `config` file for SSH.

SSH will look for the user's `~/.ssh/config` file. I have mine setup as:

    Host gitserv
        Hostname remote.server.com
        IdentityFile ~/.ssh/id_rsa.github
        IdentitiesOnly yes # see NOTES below

And I add a remote git repository:

    git remote add origin git@gitserv:myrepo.git

And then git commands work normally for me.

    git push -v origin master

**NOTES**

*   The `IdentitiesOnly yes` is required to [prevent the SSH default behavior](https://serverfault.com/questions/450796/how-could-i-stop-ssh-offering-a-wrong-key/450807#450807 "foo") of sending the identity file matching the default filename for each protocol. If you have a file named `~/.ssh/id_rsa` that will get tried BEFORE your `~/.ssh/id_rsa.github` without this option.

**References**

*   [Best way to use multiple SSH private keys on one client](https://stackoverflow.com/questions/2419566/best-way-to-use-multiple-ssh-private-keys-on-one-client)
*   [How could I stop ssh offering a wrong key](https://serverfault.com/questions/450796/how-could-i-stop-ssh-offering-a-wrong-key/450807#450807 "foo")






