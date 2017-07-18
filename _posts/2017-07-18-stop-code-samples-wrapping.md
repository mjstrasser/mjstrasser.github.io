---
layout: post
title: Stopping code samples from wrapping
categories: tech
tags: blog css
---

I needed to tweak the CSS for this blog so code samples do not wrap.  

## The goal

I wanted long lines like this one to not wrap, and be scrollable when
they are too long:

```java
    private static final Pattern REQUEST_PATTERN = Pattern.compile("^/v1/workflow/(?<crumbId>[0-9a-f]+)$");
```

They are converted by Jekyll into `pre` elements but the [Poole](http://getpoole.com)
and [Lanyon](http://lanyon.getpoole.com/) styling causes them to wrap.

## First solution

My blog stylesheet is read after those of
[Poole](http://getpoole.com) and [Lanyon](http://lanyon.getpoole.com/) and contained:

```css
pre {
  white-space: pre;
  overflow: auto;
  font-size: 0.7rem;
}
```

* `white-space: pre` prevents word wrapping
* `overflow: auto` causes too-long lines to be hidden instead of wrapped and scroll bars provided
* `font-size: 0.7em` tweaks the font a bit to my liking

This worked in Chrome and Firefox but not in Safari, which was still wrapping code.

## The Safari solution

I found that [poole.css]({{ site.baseurl }}/public/css/poole.css) contains `word-wrap: break-word`
in its `pre` rule. It appears that Safari places `word-wrap` at higher precedence than
`overflow`. 

So my [blog stylesheet]({{ site.baseurl }}/public/css/relignorant.css) now includes this for code
samples, and works in Safari as well:

```css
pre {
  white-space: pre;
  overflow: auto;
  font-size: 0.7rem;
  word-wrap: unset;
}
```

**NB:** Is *higher precedence* the correct CSS term in this case? If you know,
        [please let me know]({% link contact.md %}).
