---
layout: default
title: Web
tagline: Supporting tagline
---
{% include JB/setup %}

<!-- {% for post in site.posts %}
<div class="page-header">
  <h1>{{ post.title }} {% if post.tagline %} <small>{{ post.tagline }}</small>{% endif %}</h1>
</div>

<div class="row-fluid">
  <div class="span12">
    {{ post.content }}
  </div>
</div>
{% break %}
{% endfor %} -->

##More Posts:
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


