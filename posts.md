---
layout: default
title: "Posts"
permalink: /posts/
---

<h1>Charadata Posts</h1>

<ul>
  {% for post in site.posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a> ({{ post.date | date: "%B %d, %Y" }})</li>
  {% endfor %}
</ul>
