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

    git config --global alias.lg 'log --oneline --decorate --graph'

Example output is:

![Git log output with --decorate]({{ site.baseurl }}/public/images/git-log-decorate.png)

Then I decided to augment it with useful extra information, including colouring.

    git config --global alias.lg 'log --graph --format="%C(auto)%h %C(cyan)%cn%C(auto)%d %s %C(magenta)%cr"'

![Git log output with --format]({{ site.baseurl }}/public/images/git-log-format.png)

The `%C(auto)` specification does not colour committer name or date, so I coloured those
explicitly.

Documentation is here:

* [git config](https://git-scm.com/docs/git-config)
* [git status](https://git-scm.com/docs/git-status)
* [git log](https://git-scm.com/docs/git-log)
