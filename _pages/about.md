---
layout: about
title: About
permalink: /
subtitle: ML Engineer | Data Scientist | MSc Student at UvA

profile:
  align: right
  image: prof_pic.jpg
  image_circular: true # crops the image to make it circular
  more_info: >
    <p>Amsterdam, Netherlands</p>
    <p>University of Amsterdam</p>

selected_papers: true # includes a list of papers marked as "selected={true}"
social: true # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: true
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

I am **Masoud Rezvani** (Masoud Rezvaninejad), an ML Engineer and Data Scientist currently completing my MSc in Data Science at the University of Amsterdam (UvA). My focus is on building production-grade ML systems — reading a paper, turning it into working code, and shipping it.

I am currently an **AI Automation Intern at Talk360**, where I built an HR policy assistant using hybrid RAG (ChromaDB semantic search + BM25 keyword retrieval) reranked by a Cross-Encoder, achieving 98% faithfulness on a 22-question golden dataset. I also designed a semantic caching layer to cut repeat-query API costs, an LLM-as-judge regression pipeline (GPT-4o) for pre-deploy validation, and autonomous LangChain + Gemini agents that interpret execution logs and diagnose system failures.

Previously, as a **Data Scientist at Baly.iq** (Rocket Internet), I built XGBoost and logistic regression fraud-detection models reaching 95% accuracy, contributing to an estimated $500K in annual savings. I deployed containerised fraud-detection services with Docker and Ansible across multi-server environments, maintained AWS EC2 infrastructure (Grafana, Airflow, Netdata), and improved MySQL/PostgreSQL query performance by 20% using Percona PMM. At **Snapp!** before that, I applied CNNs and GNNs to detect fraud patterns and fraud rings, improving detection efficiency by 10%, and built the Airflow/Docker pipeline infrastructure that later went into production at Baly.iq.

My **MSc thesis** investigates the Muon optimizer — evaluating its geometry on symbolic regression loss surfaces and its behaviour in high-dimensional LLM fine-tuning via LoRA. I have a published paper in _Expert Systems with Applications_ (ESWA, October 2025): **WDAE-GAN**, a hybrid dual-autoencoder GAN with wavelet denoising for credit card fraud detection ([DOI: 10.1016/j.eswa.2025.130078](https://doi.org/10.1016/j.eswa.2025.130078)).

**Technical stack:** Python · PyTorch · HuggingFace · LangChain · MLflow · FastAPI · Docker · Airflow · AWS · ChromaDB · PostgreSQL · Rust (intermediate)
