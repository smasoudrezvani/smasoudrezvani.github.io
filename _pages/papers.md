---
layout: page
permalink: /reading-list/
title: Reading List
description: ML/AI papers I have read, am reading, or plan to read — with summaries in my own words.
nav: true
nav_order: 2
_styles: >-
  .post-header { display: none !important; }
  .papers-header { padding-top: 0.5rem; padding-bottom: 1rem; }
  .papers-header h1 { font-size: 2.2rem; }
  .filter-bar { display: flex; flex-wrap: wrap; gap: 0.4rem; margin-bottom: 0.5rem; }
  .filter-bar .btn { font-size: 0.78rem; padding: 0.25rem 0.65rem; border-radius: 999px; }
  .filter-label { font-size: 0.78rem; font-weight: 600; color: var(--global-text-color-light); align-self: center; margin-right: 0.25rem; }
  #papers-table { font-size: 0.88rem; }
  #papers-table thead th { white-space: nowrap; vertical-align: middle; }
  #papers-table tbody tr { vertical-align: middle; }
  #papers-table tbody tr.hidden { display: none; }
  .status-badge { font-size: 0.72rem; padding: 0.2rem 0.5rem; border-radius: 999px; white-space: nowrap; font-weight: 600; }
  .badge-done { background: #d1f0d1; color: #186518; }
  .badge-in-progress { background: #fff3cd; color: #7a5800; }
  .badge-to-read { background: #e9ecef; color: #495057; }
  html[data-theme="dark"] .badge-done { background: #1a4a1a; color: #7ddd7d; }
  html[data-theme="dark"] .badge-in-progress { background: #4a3800; color: #ffd966; }
  html[data-theme="dark"] .badge-to-read { background: #2e3338; color: #adb5bd; }
  .cat-badge { font-size: 0.7rem; padding: 0.15rem 0.45rem; border-radius: 4px; background: var(--global-code-bg-color); color: var(--global-text-color); font-family: monospace; }
  .expand-btn { cursor: pointer; background: none; border: none; color: var(--global-theme-color); font-size: 0.85rem; padding: 0; }
  .detail-row td { background: var(--global-code-bg-color); padding: 0.75rem 1rem; font-size: 0.82rem; }
  .detail-row.hidden { display: none; }
  .detail-row .detail-label { font-weight: 600; color: var(--global-text-color-light); margin-bottom: 0.2rem; }
  .papers-count { font-size: 0.85rem; color: var(--global-text-color-light); margin-bottom: 0.75rem; }
  .filter-bar .btn.active { font-weight: 700; }
---

<div class="post">
  <div class="papers-header">
    <h1>Reading List</h1>
  </div>

  <!-- Status filters -->
  <div class="filter-bar" id="status-filters">
    <span class="filter-label">Status:</span>
    <button class="btn btn-sm btn-outline-secondary active" data-filter-status="all">All</button>
    <button class="btn btn-sm btn-outline-success" data-filter-status="done">Done</button>
    <button class="btn btn-sm btn-outline-warning" data-filter-status="in-progress">In Progress</button>
    <button class="btn btn-sm btn-outline-secondary" data-filter-status="to-read">To Read</button>
  </div>

  <!-- Category filters -->
  <div class="filter-bar" id="cat-filters">
    <span class="filter-label">Category:</span>
    <button class="btn btn-sm btn-outline-secondary active" data-filter-cat="all">All</button>
    {% assign cats = "" | split: "" %}
    {% for paper in site.data.papers %}
      {% if paper.category != "" %}
        {% unless cats contains paper.category %}
          {% assign cats = cats | push: paper.category %}
        {% endunless %}
      {% endif %}
    {% endfor %}
    {% assign cats = cats | sort %}
    {% for cat in cats %}
      <button class="btn btn-sm btn-outline-secondary" data-filter-cat="{{ cat }}">{{ cat }}</button>
    {% endfor %}
  </div>

  <p class="papers-count">
    Showing <span id="visible-count">{{ site.data.papers | size }}</span> of {{ site.data.papers | size }} papers
  </p>

  <div class="table-responsive">
    <table class="table table-hover" id="papers-table">
      <thead>
        <tr>
          <th style="width:2rem">#</th>
          <th>Title &amp; Authors</th>
          <th style="width:5rem">Cat</th>
          <th style="width:4rem">Year</th>
          <th style="width:7rem">Status</th>
          <th style="width:2rem"></th>
        </tr>
      </thead>
      <tbody>
        {% for paper in site.data.papers %}
          {% assign row_id = forloop.index %}
          <tr class="paper-row"
              data-status="{{ paper.status }}"
              data-cat="{{ paper.category }}">
            <td class="text-muted">{{ paper.no }}</td>
            <td>
              {% if paper.link != "" %}
                <a href="{{ paper.link }}" target="_blank" rel="noopener">{{ paper.title }}</a>
              {% else %}
                {{ paper.title }}
              {% endif %}
              <br><small class="text-muted">{{ paper.authors }}{% if paper.journal != "" %} · <em>{{ paper.journal }}</em>{% endif %}</small>
            </td>
            <td>
              {% if paper.category != "" %}
                <span class="cat-badge">{{ paper.category }}</span>
              {% endif %}
            </td>
            <td class="text-muted">{{ paper.year }}</td>
            <td>
              {% if paper.status == "done" %}
                <span class="status-badge badge-done">Done</span>
              {% elsif paper.status == "in-progress" %}
                <span class="status-badge badge-in-progress">In Progress</span>
              {% else %}
                <span class="status-badge badge-to-read">To Read</span>
              {% endif %}
            </td>
            <td>
              {% if paper.why != "" or paper.summary != "" %}
                <button class="expand-btn" data-target="detail-{{ row_id }}" title="Show notes">
                  <i class="fa-solid fa-chevron-down fa-xs"></i>
                </button>
              {% endif %}
            </td>
          </tr>
          {% if paper.why != "" or paper.summary != "" %}
            <tr class="detail-row hidden" id="detail-{{ row_id }}"
                data-status="{{ paper.status }}"
                data-cat="{{ paper.category }}">
              <td></td>
              <td colspan="5">
                {% if paper.why != "" %}
                  <div class="detail-label">Why it matters</div>
                  <p>{{ paper.why }}</p>
                {% endif %}
                {% if paper.summary != "" %}
                  <div class="detail-label">Summary</div>
                  <p class="mb-0">{{ paper.summary }}</p>
                {% endif %}
              </td>
            </tr>
          {% endif %}
        {% endfor %}
      </tbody>
    </table>
  </div>
</div>

<script>
  (function () {
    var activeStatus = "all";
    var activeCat = "all";

    function applyFilters() {
      var rows = document.querySelectorAll(".paper-row");
      var visible = 0;
      rows.forEach(function (row) {
        var s = row.getAttribute("data-status");
        var c = row.getAttribute("data-cat");
        var statusOk = activeStatus === "all" || s === activeStatus;
        var catOk = activeCat === "all" || c === activeCat;
        var show = statusOk && catOk;
        row.classList.toggle("hidden", !show);
        if (show) visible++;

        // Keep detail row in sync
        var btn = row.querySelector(".expand-btn");
        if (btn) {
          var detailId = btn.getAttribute("data-target");
          var detail = document.getElementById(detailId);
          if (detail) {
            if (!show) detail.classList.add("hidden");
          }
        }
      });
      document.getElementById("visible-count").textContent = visible;
    }

    // Status filter buttons
    document.querySelectorAll("[data-filter-status]").forEach(function (btn) {
      btn.addEventListener("click", function () {
        activeStatus = this.getAttribute("data-filter-status");
        document
          .querySelectorAll("[data-filter-status]")
          .forEach(function (b) {
            b.classList.remove("active");
          });
        this.classList.add("active");
        applyFilters();
      });
    });

    // Category filter buttons
    document.querySelectorAll("[data-filter-cat]").forEach(function (btn) {
      btn.addEventListener("click", function () {
        activeCat = this.getAttribute("data-filter-cat");
        document
          .querySelectorAll("[data-filter-cat]")
          .forEach(function (b) {
            b.classList.remove("active");
          });
        this.classList.add("active");
        applyFilters();
      });
    });

    // Expand/collapse detail rows
    document.querySelectorAll(".expand-btn").forEach(function (btn) {
      btn.addEventListener("click", function () {
        var detailId = this.getAttribute("data-target");
        var detail = document.getElementById(detailId);
        var icon = this.querySelector("i");
        if (detail) {
          detail.classList.toggle("hidden");
          icon.classList.toggle("fa-chevron-down");
          icon.classList.toggle("fa-chevron-up");
        }
      });
    });
  })();
</script>
