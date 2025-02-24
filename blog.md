---
layout: default
title: Blog
permalink: /blog
nav_order: 1
---

If you use a feed reader, you can subscribe to posts via our
[Atom feed]({{ site.baseurl }}{% link feed.xml %}).

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
