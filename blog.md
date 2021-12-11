---
title: Blog
permalink: /blog
---
## Blog

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {% if page.categories %}
        <small><em>{{ page.categories | join: "</em> - <em>" }}</em></small>
      {% endif %}
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
