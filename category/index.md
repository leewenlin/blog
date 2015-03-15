---
layout: page
title: 所有分类日志
excerpt: 李文琳的日志
search_omit: true
---

{% for category in site.categories %}
{% if category.first != "articles" %}
<h4>{{category.first}}</h4>
<ul class="post-list">
{% for post in category.last %}
<li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">{{ post.excerpt }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>
{% endif %}
{% endfor %}