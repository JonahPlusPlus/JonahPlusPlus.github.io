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
        <ul>
            <li>Class: {{ class.description }}</li>
            <li>Teacher: {{ class.teacher }}</li>
            <li>Semester: {{ class.semester }}</li>
        </ul>
    </a>
{% endfor %}
</div>
