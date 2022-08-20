---
layout: default
title: Home
---
# JonahPlusPlus's Development Blog

Here you can find 

<ul>
  {% for post in site.posts %}
    <li class="post_preview">
        <a href="{{ post.url }}">{{ post.title }}</a>
        {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
