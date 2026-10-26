---
layout: default
title: Blog
permalink: /blog/
---

<h1>Blog</h1>

<h2 id="technical">Technical posts</h2>
<ul>
  {% assign technical_posts = site.posts | where: "blog_section", "technical" %}
  {% for post in technical_posts %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> ({{ post.date | date: "%Y-%m-%d" }})</li>
  {% endfor %}
</ul>

<h2 id="non-technical">Non-technical posts</h2>
<ul>
  {% assign non_technical_posts = site.posts | where: "blog_section", "non-technical" %}
  {% for post in non_technical_posts %}
    <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a> ({{ post.date | date: "%Y-%m-%d" }})</li>
  {% endfor %}
</ul>
