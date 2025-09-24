---
layout: default
title: Blog
permalink: /blog/
---

<h2>Blog Posts</h2>
<ul>
  {% for post in site.posts %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> ({{ post.date | date: "%Y-%m-%d" }})</li>
  {% endfor %}
</ul>
