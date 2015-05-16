---
layout: page
title: Technology Tidbits
description: 
---

<h2>Posts</h2>
<ul class="posts">
  {% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    {{ post.excerpt }}
  </li>

  {% endfor %}
</ul>


---
Pages created using [simple_site](http://github.com/kbroman/simple_site)
