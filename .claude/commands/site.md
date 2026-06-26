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
| `_data/papers.yml`                      | ML/AI reading list — 66 papers with category, status, why, summary        |
| `_pages/`                               | All static pages (about, notes, publications, projects, cv, series pages) |
| `_posts/`                               | All blog posts — filename format `YYYY-MM-DD-Title.md`                    |
| `_includes/`                            | Liquid partials (header.liquid, footer.liquid, etc.)                      |
| `assets/json/resume.json`               | CV data in JSONResume format — source of truth for the CV page            |
| `assets/pdf/Masoud_Rezvaninejad_CV.pdf` | CV PDF — auto-generated on every deploy by the GitHub Actions workflow    |
| `bin/update_scholar_citations.py`       | Fetches Google Scholar citation counts; requires `pyyaml` and `scholarly` |
| `requirements.txt`                      | Python deps: `nbconvert pyyaml rendercv[full] scholarly`                  |

---

## Navigation Bar

Current nav items (all title-cased, `nav: true` in frontmatter):

| Title            | File                     | nav_order            |
| ---------------- | ------------------------ | -------------------- |
| About            | `_pages/about.md`        | (home, permalink: /) |
| Book/Paper Notes | `_pages/notes.md`        | 1                    |
| Reading List     | `_pages/papers.md`       | 2                    |
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

## CV Page

The CV page (`_pages/cv.md`) uses `cv_format: jsonresume` and reads from `assets/json/resume.json`.

### resume.json structure (JSONResume schema)

Top-level keys currently used: `basics`, `education`, `work`, `publications`, `projects`, `skills`, `certificates`, `languages`, `interests`.

**Work entries (in order):**

1. Talk360 — AI Automation Intern (2026-04 → Present)
2. Baly.iq (Rocket Internet) — Data Scientist (2024-02 → 2025-11)
3. Snapp! (Rocket Internet) — Data Analyst (2020-09 → 2024-02)

**Certifications:** 7 entries (Udemy AI Engineer, ByteByteGo ML System Design, Stanford CME295, Stanford CS336, Arvancloud DevOps, Coursera Algorithms, Udemy R stats).

**Projects in resume.json:** DocVQA, SemArt, ML Interpretation Dashboard, Algorithmic Trading System, LLM Arabic Fraud Detection, CARE-GNN Reconstruction.

### Dynamic PDF generation

`assets/pdf/Masoud_Rezvaninejad_CV.pdf` is **auto-generated on every deploy** — do NOT manually edit this PDF. The mechanism lives in `.github/workflows/deploy.yml` as a "Generate CV PDF" step between the CSS purge and the deploy:

1. Installs Playwright + Chromium
2. Serves `_site` locally on port 4000
3. Playwright renders `http://localhost:4000/cv/` and saves the PDF directly to `_site/assets/pdf/`
4. Deploy action pushes `_site` to GitHub Pages (including the fresh PDF)

No commit-back loop: the PDF is written to the built `_site` output, not back to the source branch.

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
- **Tables** — write standard markdown tables. Bootstrap classes (`.table .table-bordered .table-striped`) are applied automatically by a JavaScript snippet in `_layouts/post.liquid`. **Do NOT add `{: .table ...}` IAL** — Prettier always inserts a blank line before it, which breaks kramdown and makes borders invisible.

Posts created with the help of NotebookLM should include two block-tip callouts after the intro:

```markdown
> ##### Source
>
> Notes drawn from Chapter N of _Book Title_ by Author.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }
```

---

## Series Architecture

Hub page at `/notes/` (`_pages/notes.md`) lists all series from `_data/series.yml`.

Each series has a dedicated page at `/notes/<series-id>/` that filters posts with:

```liquid
{% assign series_posts = site.posts | where: 'series', '<id>' | sort: 'path' | sort: 'date' %}
```

**Why two sorts?** `sort: 'date'` alone breaks when multiple posts share the same date (e.g. all parts published on the same day). Liquid's sort is stable, so sorting by `path` (filename) first, then by `date`, ensures equal-date posts stay in filename/alphabetical order — which is the intended Part 1 → Part 2 → … order. **Never use `sort: 'date'` alone on a series page.**

**The two-digit part-number trap:** When two posts share the same date AND their part numbers cross a power of ten (e.g. "Part 9" and "Part 10"), alphabetical path sort puts "Part 10" before "Part 9" because `'1' < '9'`. The date sort then preserves that wrong order. **Fix: give the higher-numbered post a strictly later date** (e.g. Part 9 → `2026-06-15`, Part 10 → `2026-06-16`). This ensures the date sort correctly separates them regardless of alphabetical path order. This issue recurs every time a series crosses from single-digit to double-digit (Part 9 → 10) or double-digit decades (Part 19 → 20).

### Current Series

| id                       | Title                                 | Source                          | Page                                      |
| ------------------------ | ------------------------------------- | ------------------------------- | ----------------------------------------- |
| `reinforcement-learning` | Reinforcement Learning                | Sutton & Barto (2018)           | `_pages/series-reinforcement-learning.md` |
| `ml-papers`              | ML Papers                             | Standalone deep-dives           | `_pages/series-ml-papers.md`              |
| `game-theory`            | Game Theory                           | Osborne (2003)                  | `_pages/series-game-theory.md`            |
| `ddia`                   | Designing Data-Intensive Applications | Kleppmann & Riccomini (2nd ed.) | `_pages/series-ddia.md`                   |

### Adding a New Series

1. Add entry to `_data/series.yml` (id, title, subtitle, description, icon, url)
2. Create `_pages/series-<id>.md` with permalink `/notes/<id>/`, `nav: false`
   - Use `sort: 'path' | sort: 'date'` — never `sort: 'date'` alone (see note above)
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

### Designing Data-Intensive Applications series (`series: ddia`) — based on Kleppmann & Riccomini (2nd ed.)

Notes created with the help of NotebookLM from podcast-style audio overviews of the book chapters.

- `2026-06-04-DDIA Part 1 - Trade-Offs in Data Systems Architecture.md`
- `2026-06-04-DDIA Part 2 - Defining Nonfunctional Requirements.md`
- `2026-06-04-DDIA Part 3 - Data Models and Query Languages.md`
- `2026-06-04-DDIA Part 4 - Storage and Retrieval.md`
- `2026-06-04-DDIA Part 5 - Encoding and Evolution.md`
- `2026-06-06-DDIA Part 6 - Replication.md`
- `2026-06-06-DDIA Part 7 - Sharding.md`
- `2026-06-06-DDIA Part 8 - Transactions.md`
- `2026-06-15-DDIA Part 9 - The Trouble with Distributed Systems.md`
- `2026-06-15-DDIA Part 10 - Consistency and Consensus.md` (frontmatter date: 2026-06-16 — bumped to sort after Part 9)
- `2026-06-26-DDIA Part 11 - Batch Processing.md`
- `2026-06-26-DDIA Part 12 - Stream Processing.md`
- `2026-06-26-DDIA Part 13 - A Philosophy of Streaming Systems.md`
- `2026-06-26-DDIA Part 14 - Doing the Right Thing.md`

---

## Existing Projects (`_projects/`)

| File                | Title                       | importance | Notes                                 |
| ------------------- | --------------------------- | ---------- | ------------------------------------- |
| `1_docvqa.md`       | Multimodal DocVQA           | 1          | img: assets/img/docvqa_preview.png    |
| `2_semart.md`       | SemArt Digital Heritage     | 2          | img: assets/img/semart_preview.png    |
| `3_ml_dashboard.md` | ML Interpretation Dashboard | 3          | img: assets/img/dashboard_preview.png |
| `4_llm_fraud.md`    | LLM Fraud Detection         | 4          | img: assets/img/fraud_preview.png     |
| `5_s3_minio.md`     | S3-MinIO Starter Kit        | 5          | img: assets/img/minio_preview.png     |
| `6_care_gnn.md`     | CARE-GNN Reconstruction     | 6          | no img (source image not available)   |
| `7_algo_trading.md` | Algorithmic Trading System  | 7          | no img (source image not available)   |

**Important:** Project `img:` fields must reference files that actually exist in `assets/img/`. The build uses imagemagick to generate `-800.webp` and `-1400.webp` variants. If the source file is missing, the build produces broken WebP links that fail the `check-links-on-site` CI check. Omit `img:` rather than pointing to a non-existent file.

Available project preview images in `assets/img/`: `docvqa_preview.png`, `semart_preview.png`, `dashboard_preview.png`, `fraud_preview.png`, `minio_preview.png` (and generic: `1.jpg`–`12.jpg`, `rhino.png`, `prof_pic.jpg`).

---

## Pre-Commit Checklist

1. **Run Prettier** (mandatory — always, no exceptions): `npx prettier . --write`
2. Build locally: `docker compose up --build`
3. Check http://localhost:8080 — navigation, pages, dark mode
4. Never commit: data files, model weights, `.env`

### Prettier line-ending side-effect (Windows)

Running `npx prettier . --write` on Windows normalises line endings from LF to CRLF across all touched files. This causes VS Code to show many existing posts as modified (orange **M**) even though **no actual content changed**. `git diff` will show only `LF will be replaced by CRLF` warnings with no content diff lines. This is expected — **do commit these files** (git normalises back to LF in the object store; Linux CI sees clean files). The CI Prettier check passes because it runs on Linux which produces LF.

---

## Automated Steps When Using This Skill

> **IMPORTANT:** After creating or editing ANY post, page, or data file using this skill,
> you MUST run `npx prettier . --write` before finishing. This is not optional —
> the site's CI pipeline enforces Prettier formatting and will reject non-conforming files.
> Run it unconditionally; it is safe to run on unchanged files (they will be reported as `unchanged`).

---

## Common Tasks (Quick Reference)

**Publish a new post:**

1. Create `_posts/YYYY-MM-DD-Title.md` with correct frontmatter (series + categories fields).
2. Run `npx prettier . --write`.

**Add a nav page:**

1. Add `nav: true` + `nav_order: N` + capitalized `title:` to the page's frontmatter.
2. Run `npx prettier . --write`.

**Add a new project:**

1. Create `_projects/N_slug.md` with `importance: N` and `category: work`.
2. Only set `img:` if the image file actually exists in `assets/img/` — omit otherwise.
3. Run `npx prettier . --write`.

**Update the CV:**

1. Edit `assets/json/resume.json` (JSONResume format).
2. The PDF at `assets/pdf/Masoud_Rezvaninejad_CV.pdf` is auto-regenerated on the next deploy — do not edit it manually.
3. Run `npx prettier . --write`.

**Change social links:**
Edit `_data/socials.yml` — changes take effect after rebuild. Run `npx prettier . --write`.

**Add a new series:**
Edit `_data/series.yml` + create `_pages/series-<id>.md`. Run `npx prettier . --write`.
