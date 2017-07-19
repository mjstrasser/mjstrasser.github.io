---
layout: post
title: Announcing blog posts on Twitter using IFTTT
category: tech
tags: blog
---

When you have a blog you usually want to let your followers know when
a new article is posted. This blog has [an Atom feed]({{ site.baseurl }}{% link atom.xml %})
but RSS readers are less common than they used to be and seem a bit old fashioned.

* TOC
{:toc}

## Announce on Twitter

Announcing a new blog post on [my Twitter account](https://twitter.com/pharsicle)
seems like the modern way to go. But how?

A long-term solution will probably be to use a post-commit or post-push hook in
[GitHub where this blog is hosted](https://github.com/mjstrasser/mjstrasser.github.io).

## Quick solution: IFTTT

I remembered that I have an account at [If This Then That](https://ifttt.com/) and decided
to try using that. It is already linked to my Twitter account so that part of the process
was ready to go.
I created a new Applet (IFTTT-speak for a set of rules that link services together;
formerly called a Recipe) to check the Atom feed periodically and tweet an announcement.

But IFTTT showed the Applet as _never run_ and did not display any error messages that I
could find.

## Invalid Atom feed

IFTTT help pages include a link to the [W3C Feed Validation Service](http://validator.w3.org/feed/).
Using that showed the feed to be invalid because the email address field was empty.
This was solved by adding `author.email` to `_config.yml`. (I used a valid
email address that I can get [my mail provider](https://fastmail.com/) to filter if
it starts attracting spam.)

There were other, minor issues, including the `link` information showing
`http://blog.michaelstrasser.com` instead of `https://blog.michaelstrasser.com`. I don’t know
why Jekyll does that, but a solution is to add an explicit `url` value to the configuration
file:

```yaml
url:  https://blog.michaelstrasser.com
```

(This value is only used in [Jekyll production
mode](https://jekyllrb.com/docs/configuration/#specifying-a-jekyll-environment-at-build-time).
In the default development mode it is overwritten by the actual URL: `http://localhost:4000`
by default.)

## Success

**Update:** The tweet for this page is:

![Example tweet sent by IFTTT]({{ site.baseurl }}/public/images/tweet-pharsicle-ifttt.png)

By default, IFTTT uses its own link shortener, but I turned that off [as described
here](https://michaelsoolee.com/ifttt-links/).
