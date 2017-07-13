---
layout: post
title: Showing tags on posts
categories: tech
tags: blog
---

After adding a 
[Categories page]({{ site.baseurl }}{% post_url 2017-07-13-adding-a-categories-page %})
I wanted to add tag-handling to the blog. I wanted to display the tags assigned
to each post under the title.

Using the same principles as for categories and adapting some CSS from
[Codinfox](https://codinfox.github.io/dev/2015/03/06/use-tags-and-categories-in-your-jekyll-based-github-pages/),
I have achieved that. Each tag name is a link to the relevant section of the
[Tags page]({{ site.baseurl }}{% link tags.html %}).

Like categories, tags are specified for a post in the front matter, as for this post:

```yaml
---
layout: post
title: Showing tags on posts
categories: tech
tags: blog
---
```
