---
layout: page
title: Memos
tagline: 
---
{% include JB/setup %}


{% for post in site.posts limit:5  %}
<article class="post">
  <h3 class="title"><a href="{{ BASE_PATH }}{{post.url}}">{{ post.title }}</a>
    <span class="date">{{ post.date | date_to_string }}</span>
  </h3>
  <section class="muted">{{ post.excerpt }} <a href="{{ BASE_PATH }}{{post.url}}" class="more">...more</a></section>
</article>
{% endfor %}

