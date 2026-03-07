---
layout: default
title: "Blog"
permalink: /blog/
---

## All Posts

{% for post in site.posts %}
- **[{{ post.title }}]({{ post.url | relative_url }})** -- {{ post.date | date: "%B %-d, %Y" }}{% if post.description %}<br>{{ post.description }}{% endif %}
{% endfor %}
