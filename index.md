---
layout: default
title: Home
---
# JonahPlusPlus's Development Blog

Here you can find posts on some of my work.

{% include heading.html heading="Posts" %}

{% for post in site.posts %}
    <div class="post_preview">
        <a href="{{ post.url }}">{{ post.title }}</a>
        {{ post.excerpt }}
    </div>
{% endfor %}
