---
layout: default
title: Home
---

# Nate Maxwell's Blog

Welcome to my blog! Here are all my posts:

## Post History

<ul>
{% assign sorted_posts = site.posts | sort: 'date' | reverse %}
{% for post in sorted_posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    {% if post.date %}
      - *{{ post.date | date: "%-m/%-d/%y" }}*
    {% endif %}
    {% if post.categories %}
      — {{ post.categories | join: ', ' }}
    {% endif %}
  </li>
{% endfor %}
</ul>
