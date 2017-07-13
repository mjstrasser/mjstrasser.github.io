---
layout: post
title: Adding a categories page to this blog
categories: tech
tags: blog
---

I succeeded in adding a [Categories page]({{ site.baseurl }}{% link categories.html %}) to
this blog.

Jekyll supports specification of variables `category`, `categories` and `tags` in the front
matter of blog posts but it is left as an Exercise for the Reader™ to do anything with them. The
[Lanyon](http://lanyon.getpoole.com/) theme does not itself include page with category or tag lists
(but of course is infinitely expandable).

So I created a contents page by adapting [this
technique](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/)
with my own amateur CSS applied. The page is `contents.html` in the root directory of the
site and has front matter:

```yaml
---
layout: page
title: Categories
---
```

The [Lanyon](http://lanyon.getpoole.com/) sidebar automatically creates links to any
pages with `layout: page` in the front matter so the Categories page is easily found. 
