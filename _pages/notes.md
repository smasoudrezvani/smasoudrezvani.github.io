---
layout: page
permalink: /notes/
title: Book/Paper Notes
description: Structured reading notes organized by book and topic series.
nav: true
nav_order: 1
---

<div class="post">

  <div class="header-bar">
    <h1>Book/Paper Notes</h1>
    <h2>Structured notes organized by book and topic series</h2>
  </div>

  <ul class="post-list mt-4">
    {% for series in site.data.series %}
      {% assign series_posts = site.posts | where: "series", series.id | sort: "date" %}
      <li>
        <div class="d-flex justify-content-between align-items-start">
          <h3 class="mb-0">
            <i class="{{ series.icon }}"></i>&nbsp;
            <a class="post-title" href="{{ series.url | relative_url }}">{{ series.title }}</a>
          </h3>
          <span class="post-meta ml-2" style="white-space: nowrap;">{{ series_posts.size }} posts</span>
        </div>
        <p class="post-meta mb-1">{{ series.subtitle }}</p>
        <p>{{ series.description }}</p>
        <a href="{{ series.url | relative_url }}" class="btn btn-sm btn-outline-primary series-btn">
          View Series &nbsp;<i class="fa-solid fa-arrow-right fa-sm"></i>
        </a>
      </li>
    {% endfor %}
  </ul>

</div>
