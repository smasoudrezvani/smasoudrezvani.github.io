# Site Knowledge Base — smasoudrezvani.github.io

You are helping Masoud Rezvaninejad maintain his personal academic website.
Load this full context before taking any action on the site.

---

## Tech Stack

- **Theme:** al-folio (Jekyll) — responsive academic theme
- **Hosting:** GitHub Pages (branch: main, URL: https://smasoudrezvani.github.io)
- **Local dev:** Docker (`docker compose up` → http://localhost:8080)
- **Formatter:** Prettier (`npx prettier . --write`) — must pass before every commit
- **Social icons:** jekyll-socials plugin v0.0.6 — config in `_data/socials.yml`

---

## Key Files & Directories

| Path                                    | Purpose                                                                   |
| --------------------------------------- | ------------------------------------------------------------------------- |
| `_config.yml`                           | Global Jekyll config (url, baseurl, plugins)                              |
| `_data/socials.yml`                     | Social media links (CV, email, LinkedIn, Scholar, GitHub)                 |
| `_data/series.yml`                      | Series metadata (id, title, subtitle, description, icon, url)             |
| `_pages/`                               | All static pages (about, notes, publications, projects, cv, series pages) |
| `_posts/`                               | All blog posts — filename format `YYYY-MM-DD-Title.md`                    |
| `_includes/`                            | Liquid partials (header.liquid, footer.liquid, etc.)                      |
| `assets/pdf/Masoud_Rezvaninejad_CV.pdf` | CV PDF                                                                    |

---

## Navigation Bar

Current nav items (all title-cased, `nav: true` in frontmatter):

| Title            | File                     | nav_order            |
| ---------------- | ------------------------ | -------------------- |
| About            | `_pages/about.md`        | (home, permalink: /) |
| Book/Paper Notes | `_pages/notes.md`        | 1                    |
| Publications     | `_pages/publications.md` | 3                    |
| Projects         | `_pages/projects.md`     | 4                    |
| CV               | `_pages/cv.md`           | 5                    |

Series pages (`nav: false`) live under `/notes/<series-id>/`.

---

## Social Links (`_data/socials.yml`)

```yaml
cv_pdf: /assets/pdf/Masoud_Rezvaninejad_CV.pdf
email: s.masoudrezvani@gmail.com
linkedin_username: smasoudrezvani
scholar_userid: V1flNroAAAAJ
github_username: smasoudrezvani
```

Supported keys come from jekyll-socials plugin. To add a platform not in the plugin,
use `custom_social: { logo: <url>, title: <name>, url: <link> }`.

---

## Blog Post Frontmatter Template

```yaml
---
layout: post
title: "Part N: <Title>"
date: YYYY-MM-DD
tags: tag1 tag2
categories: <series-id> # matches series id for category archive
series: <series-id> # used by series hub pages to filter posts
mermaid:
  enabled: true
  zoomable: true
---
```

Supported rich content in posts:

- **LaTeX math** — inline `$...$`, block `$$...$$`
- **Mermaid diagrams** — fenced ```mermaid blocks (enable in frontmatter)
- **ECharts** — interactive charts via ```echarts blocks
- **Block tips** — `{: .block-tip}`, `{: .block-warning}`, `{: .block-danger}`
- **Tables** — use `.table .table-bordered .table-striped` classes

---

## Series Architecture

Hub page at `/notes/` (`_pages/notes.md`) lists all series from `_data/series.yml`.

Each series has a dedicated page at `/notes/<series-id>/` that filters posts with:

```liquid
{% assign series_posts = site.posts | where: 'series', '<id>' | sort: 'date' %}
```

### Current Series

| id                       | Title                  | Source                | Page                                      |
| ------------------------ | ---------------------- | --------------------- | ----------------------------------------- |
| `reinforcement-learning` | Reinforcement Learning | Sutton & Barto (2018) | `_pages/series-reinforcement-learning.md` |
| `ml-papers`              | ML Papers              | Standalone deep-dives | `_pages/series-ml-papers.md`              |
| `game-theory`            | Game Theory            | Osborne (2003)        | `_pages/series-game-theory.md`            |

### Adding a New Series

1. Add entry to `_data/series.yml` (id, title, subtitle, description, icon, url)
2. Create `_pages/series-<id>.md` with permalink `/notes/<id>/`, `nav: false`
3. New posts: add `series: <id>` and `categories: <id>` to frontmatter

---

## Existing Posts

### Reinforcement Learning series (`series: reinforcement-learning`)

- `2026-04-01-RL intro.md` — Part 1: Comprehensive Intro to RL
- `2026-04-02-Armed Bandit.md` — Part 2: Multi-Armed Bandit
- `2026-04-03-Tabular methods.md` — Part 3: Tabular Methods
- `2026-04-06-RL stability tests.md`
- `2026-04-06-Sarsa vs. qlearning.md`
- `2026-04-07-Function approximation.md`
- `2026-04-08-Policy Gradient.md`
- `2026-04-08-PPO, GRPO, and DPO.md`

### Game Theory series (`series: game-theory`) — based on Osborne (2003)

- `2026-04-20-Game theory intro.md` — overview of all topics
- `2026-04-21-Nash Equilibrium.md`
- `2026-04-21-Mixed Strategies.md`
- `2026-04-21-Extensive Games.md`
- `2026-04-21-Bayesian Games.md`
- `2026-04-21-Bargaining and Coalitions.md`

### ML Papers series (`series: ml-papers`)

- `2026-03-30-muon optimizer.md`
- `2026-03-31-Causal Inference and Impact Evaluation.md`

---

## Pre-Commit Checklist

1. Run Prettier: `npx prettier . --write`
2. Build locally: `docker compose up --build`
3. Check http://localhost:8080 — navigation, pages, dark mode
4. Never commit: data files, model weights, `.env`

---

## Common Tasks (Quick Reference)

**Publish a new post:**
Create `_posts/YYYY-MM-DD-Title.md` with correct frontmatter (series + categories fields).

**Add a nav page:**
Add `nav: true` + `nav_order: N` + capitalized `title:` to the page's frontmatter.

**Change social links:**
Edit `_data/socials.yml` — changes take effect after rebuild.

**Add a new series:**
Edit `_data/series.yml` + create `_pages/series-<id>.md`.
