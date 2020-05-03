---
layout: page
title: Home
permalink: /
---

Welcome to my blog site, where I'll post security write-ups, audit reports, tutorials, etc.

Hope you enjoy what you read.

See the [about](/about/) page for contact information.

### Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
