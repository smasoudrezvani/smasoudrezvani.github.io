---
layout: page
permalink: /notes/ml-papers/
title: "ML Papers"
description: "In-depth notes on individual machine learning papers and techniques"
nav: false
---

<div class="post">

  <p class="text-muted mb-3">
    <i class="fa-solid fa-file-lines"></i>&nbsp;
    Standalone deep-dives into individual ML papers and techniques
  </p>

  <a href="{{ '/notes/' | relative_url }}" class="btn btn-sm btn-outline-secondary mb-4">
    <i class="fa-solid fa-arrow-left fa-sm"></i>&nbsp; All Series
  </a>

  {% assign series_posts = site.posts | where: "series", "ml-papers" | sort: "date" | reverse %}

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
