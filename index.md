---
layout: default
title: Jack Dintruff
---

{% for post in site.posts %}
  <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
  <img src="{{ post.image }}" alt="{{ post.title }} Preview Image">
  <p>{{ post.excerpt }}</p>
  <hr>
{% endfor %}
