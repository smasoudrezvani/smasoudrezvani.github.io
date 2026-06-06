---
layout: page
permalink: /notes/ddia/
title: "Designing Data-Intensive Applications"
description: "Chapter notes based on Designing Data-Intensive Applications (2nd ed.) by Martin Kleppmann & Chris Riccomini"
nav: false
---

<div class="post">

  <p class="text-muted mb-3">
    <i class="fa-solid fa-database"></i>&nbsp;
    Based on <em>Designing Data-Intensive Applications</em> (2nd ed.) by Martin Kleppmann &amp; Chris Riccomini
  </p>

  <a href="{{ '/notes/' | relative_url }}" class="btn btn-sm btn-outline-secondary mb-4">
    <i class="fa-solid fa-arrow-left fa-sm"></i>&nbsp; All Series
  </a>

{% assign series_posts = site.posts | where: "series", "ddia" | sort: "path" | sort: "date" %}

  <ul class="post-list">
    {% for post in series_posts %}
      {% assign read_time = post.content | number_of_words | divided_by: 180 | plus: 1 %}
      <li>
        <h3>
          <a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </h3>
        {% if post.description %}<p>{{ post.description }}</p>{% endif %}
        <p class="post-meta">
          {{ read_time }} min read &nbsp;&middot;&nbsp;
          {{ post.date | date: '%B %d, %Y' }}
        </p>
      </li>
    {% endfor %}
  </ul>

</div>
