---
layout: post
title: Putting tags into HTML meta keywords
category: tech
tags: blog
---

I wanted to include a page’s tags in an HTML `meta` tag as keywords.

Given a page with these tags:

<span class="post-tag">gradle</span>
<span class="post-tag">spring-boot</span>

I want the HTML header to contain:

```html
<meta name="keywords" content="gradle,spring-boot"> 
```

I achieved that simply by adding this into the `head.html` include file:

```html
{% raw %}
  {% if page.tags %}
  <meta name="keywords" content="{{ page.tags | join: ',' }}">
  {% endif %}
{% endraw %}
```
