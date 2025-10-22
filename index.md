---
layout: default
title: Home
---

# About Me

I'm an art production pipeline engineer and tech artist. I've been bouncing
back and forth between games and films for several years now.

I've worked on several games as an unreal technical artist, such as The Quarry,
Crossfire X, and several unannounced games.

I often work on real-time 'adjacent' projects that aren't quite games but are
still entertainment experiences using game engines such as the award-winning
VR exhibit: Time Magazine's The March.

Additionally, I've architected and helped build a previz + postviz + techviz +
virtual production pipeline at Digital Domain 3.0.

Here is my [work history](https://github.com/nate-maxwell)
(if I remember to update it) and my [resume](https://drive.google.com/file/d/1LbOymKKA-_lS2Jjo5zvysWKhy0_TtTVI/view?usp=sharing),
as well as my [linkedin](https://www.linkedin.com/in/nathandmaxwell/).

# Post History

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
