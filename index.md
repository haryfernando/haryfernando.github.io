---
layout: default
title: hary.my.id 
lang: en
---
<div id="main_content_wrap" class="outer">
<section id="main_content" class="inner">
{% for post in site.categories.en %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  {% if post.content contains "<!-- more -->" %}
   {{ post.content | split:"<!-- more -->" | first % }}
  {% else %}
   {{ post.content | strip_html | truncatewords:100 }}
  {% endif %}
{% endfor %}
</section>
</div>
