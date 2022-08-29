---
layout: page
title: Home
---
<h1 id="title">JonahPlusPlus's Development Blog</h1>

Here you can find posts on some of my work.

{% include heading.html heading="Posts" %}

<div>
{% for post in site.posts %}
    <div class="post_preview">
        <a href="{{ post.url }}"><h3>{{ post.title }}</h3></a>
        {{ post.excerpt }}
    </div>
{% endfor %}
</div>
