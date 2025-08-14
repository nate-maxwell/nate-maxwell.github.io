---
layout: home
title: Home
---

# Nate Maxwell's Blog

Welcome to my blog! Here are all my posts:

## Post History

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) - *{{ post.date | date: "%m/%d/%y" }}*
{% endfor %}
