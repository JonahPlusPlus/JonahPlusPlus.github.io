---
layout: page
title: Portfolio
---
<h1 id="title">Portfolio</h1>

{% include heading.html heading="Classwork" %}

<div>
{% for class in site.classes %}
    <a href="{{ class.url }}" class="card">
        <h3>{{ class.title }}</h3>
        <p>{{ class.excerpt | strip_html }}</p>
    </a>
{% endfor %}
</div>
