---
layout: page
title: Lasted Post
tagline: index
---
{% include JB/setup %}

{% for post in site.posts %}
  <h1 class="emphnext">{{ post.title }}</h1>
  {{ post.content }}
  {% break %}
{% endfor %}

Other Posts:
--------------
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {% if forloop.index > 9 %}
      <li><span>More Posts</span> &raquo; <a href="{{ BASE_PATH }}/archive.html" title="archive">Archive</a> </li> 
      {% break %}
    {% endif %}
  {% endfor %}
</ul>