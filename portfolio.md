---
layout: page
title: Portfolio
---
<h1 id="title">Portfolio</h1>

<div>
{% for class in site.classes %}
    <div class="post_preview">
        <a href="{{ class.url }}"><h3>{{ class.title }}</h3></a>
        {{ class.excerpt }}
    </div>
{% endfor %}
</div>
