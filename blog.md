---
layout: page
title: Blog
---
<h1 id="title">Blog</h1>

{% include heading.html heading="Posts" %}

<div>
{% for post in site.posts %}
    <a href="{{ post.url }}" class="card">
        <h3>{{ post.title }}</h3>
        <p>{{ post.excerpt }}</p>
    </a>
{% endfor %}
</div>
