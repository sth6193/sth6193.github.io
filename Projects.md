---
layout: page
title: Projects
date: 2019-21-7 12:00:00
---

# Personal Projects

*Small things I've worked on outside of my graduate studies.*


{% for post in site.categories.Projects %}

<div class="post py3">
  <p class="post-meta">
{% if site.date_format %}
{{ post.date | date: site.date_format }}
{% else %}
{{ post.date | date: "%b %-d, %Y" }}
{% endif %}
</p>
  <a href="{{ post.url | relative_url }}" class="post-link"><h3 class="h1 post-title">{{ post.title }}</h3></a>
  <span class="post-summary">
    {% if post.summary %}
      {{ post.summary }}
    {% else %}
      {{ post.excerpt }}
    {% endif %}
  </span>
</div>

{% endfor %}
