---
layout: page
title: Archive
---

# Archive

{% for post in site.posts %}
<li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
