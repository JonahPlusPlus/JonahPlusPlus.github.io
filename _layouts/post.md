---
layout: default
---

<h1 id="title">{{ page.title }}</h1>

{% if page.image and page.image != "" and page.image != nil %}
	{% include image.html image=page.image caption=page.caption %}
{% endif %}

{{ content }}
