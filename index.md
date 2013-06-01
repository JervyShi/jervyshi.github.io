---
layout: default
title: Index
---
{% include JB/setup %}

{% for post in site.posts %}
{% if forloop.first %}
<div class="page-header">
  <h1>{{ post.title }}</h1>
</div>
<div class="row-fluid">
  <div class="span12">
    {{ post.content }}
  </div>
</div>
{% endif %}
{% endfor %}

<hr/>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>