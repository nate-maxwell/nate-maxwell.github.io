---
layout: default
title: Home
---

## About me

I'm an art production pipeline engineer and tech artist. I've been bouncing
back and forth between games and films for several years now.

I've worked on several games as an unreal technical artist and built an entire
previz / postviz / techviz / virtual production pipeline at Digital Domain 3.0.

You can see my film work history [here](https://github.com/nate-maxwell) and
view my resume
[here](https://drive.google.com/file/d/11D7EMN3QdCTIFKhBteWEr4Emn7nQPYvG/view?usp=sharing)

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
