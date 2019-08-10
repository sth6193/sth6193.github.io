---
layout: page
title: Projects
date: 2019-21-7 12:00:00
---

{% for post in site.categories.Projects %}
 <li><span>{{ post.date | date_to_string }}</span> &nbsp; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
