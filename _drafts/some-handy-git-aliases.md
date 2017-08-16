---
layout: post
title: Some handy Git aliases
category: tech
tags: git
---

I got some useful Git aliases a while ago and was starting with a new client where I
needed to recreate them so I decided the document them here.

`git s` is useful for short status display, defined like this:

    git config --global alias.s 'status --short'

`git lg` is a short, graphical log display, defined like this:

    git config --global alias.lg 'log --oneline --decorate --all --graph'

Example output is:

![Git log output with --decorate --all]({{ site.baseurl }}/public/images/git-log-decorate-all.png)

I remade it with extra information, that include committer name and date.

    git config --global alias.lg 'log --oneline --graph --format="%C(auto)%h [%cn] %d %s [%cr]"'

![Git log output with --format]({{ site.baseurl }}/public/images/git-log-format.png)

Documentation is here:

* [git config](https://git-scm.com/docs/git-config)
* [git status](https://git-scm.com/docs/git-status)
* [git log](https://git-scm.com/docs/git-log)
