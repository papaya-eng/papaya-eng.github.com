---
layout: default
title: index
---
 
<section class="content">
  <ul class="entries">
    {% for post in site.posts limit:8 %}
    <li>
      <a href="{{ post.url }}">
        <h3>{{ post.title }}</h3>
      </a>
      <p>{{ post.content }}</p>
    </li>
    {% endfor %}
  </ul>
</section>