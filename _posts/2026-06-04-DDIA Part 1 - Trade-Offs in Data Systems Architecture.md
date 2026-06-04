---
layout: post
title: "Part 1: Trade-Offs in Data Systems Architecture"
date: 2026-06-04
tags: ddia data-systems oltp olap cloud distributed-systems
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

The very first lesson of serious systems engineering is that there is no perfect architecture. As the economist Thomas Sowell put it: _"There are no solutions, there are only trade-offs."_ Every technology choice — from the database you pick to how you deploy your service — involves accepting costs in exchange for benefits. Chapter 1 of DDIA (2nd ed.) frames the entire book around this principle.

> ##### Source
>
> Notes drawn from Chapter 1 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. The Two Audiences for Data

Modern systems serve two fundamentally different classes of users, each with radically different access patterns:

| Audience                  | Goal                               | Pattern                                   |
| ------------------------- | ---------------------------------- | ----------------------------------------- |
| **End users**             | Interact with the live application | High-frequency, low-latency point lookups |
| **Analysts / scientists** | Understand trends in the business  | Rare, expensive scans over large datasets |

{: .table .table-bordered .table-striped}

Trying to serve both audiences with the same database is one of the most common early-stage performance mistakes.

---

## 2. OLTP vs OLAP

### Online Transaction Processing (OLTP)

OLTP databases power customer-facing applications. Think of PostgreSQL backing a web shop: every login, cart update, and purchase is a small, targeted operation against a handful of rows.

**Under the hood:**

- Data is laid out **row by row** on disk (row-oriented storage).
- Queries navigate directly to the relevant row via a **B-tree index** — the database never has to scan the whole table.
- Optimised for **low-latency point reads and writes** (single-digit milliseconds).

### Online Analytical Processing (OLAP)

Business analysts and data scientists ask questions like _"What was revenue per region in Q4 across all 50 stores?"_ — queries that scan millions of rows but only read a handful of columns.

Running such a query against a live OLTP database would pin the CPU at 100% and freeze the application for every real user. The solution is a **separate, read-only data warehouse**.

**Under the hood:**

- Data is loaded via an nightly **ETL** (Extract–Transform–Load) process.
- The warehouse uses **columnar storage**: all values for one column (e.g., all prices) are stored together on disk.
- When the analyst asks for the average price, the database reads only the price column into the CPU — never the name, address, or email of each customer.
- Combined with SIMD vector processing, columnar layout can aggregate billions of rows in seconds.

```mermaid
graph LR
    A[Live App] -->|point queries| B[(OLTP DB)]
    B -->|nightly ETL| C[(Data Warehouse)]
    C -->|analytical queries| D[Analysts / BI]

    classDef default fill:#1a1a1a,stroke:#888,stroke-width:2px,color:#fff;
    classDef highlight fill:#2196F3,stroke:#fff,stroke-width:2px,color:#fff;
    class A,D highlight;
```

---

## 3. Data Lakes: The Schema-on-Read Revolution

A data warehouse works beautifully for questions you already know how to ask. But machine learning requires raw, messy data — unstructured JSON logs, audio files, IoT sensor readings — that doesn't fit a rigid SQL schema.

|                        | Data Warehouse                       | Data Lake                                  |
| ---------------------- | ------------------------------------ | ------------------------------------------ |
| **Schema enforcement** | Schema-on-write (rejected at ingest) | Schema-on-read (interpreted at query time) |
| **Typical storage**    | Columnar files (Parquet, ORC)        | Raw files on object storage (S3, GCS)      |
| **Best for**           | Known analytical queries             | Exploratory ML, NLP, feature engineering   |

{: .table .table-bordered .table-striped}

The practical rule: a data warehouse answers questions you already understand; a data lake holds raw material for questions you haven't thought of yet. Both often coexist in the same organisation.

---

## 4. Systems of Record vs. Derived Data

Every non-trivial system separates these two roles:

- **System of record** — the authoritative, permanent source of truth (e.g., the primary PostgreSQL database). Optimised for safe, consistent writes.
- **Derived data system** — a redundant copy optimised for a specific access pattern (e.g., a Redis cache, a search index, a materialised view). It can always be rebuilt from the source of record.

The hardest problem this introduces is **cache invalidation**: ensuring derived copies are updated when the source of truth changes. This is a fundamental tension that appears throughout the book.

---

## 5. Cloud vs. Self-Hosting

### The Self-Hosting Problem: Capacity Planning

Owning bare-metal servers forces a brutal binary choice:

- **Under-provision** → your app crashes on a viral traffic spike.
- **Over-provision** → you pay for idle hardware 99% of the time.

### Cloud Elasticity

Cloud providers virtualise hardware and let you rent compute by the second. If traffic spikes, you spin up 100 extra VMs in milliseconds (**horizontal scaling**). When traffic drops, you terminate them and stop paying.

The modern operations role (SRE / DevOps) shifted from physically racking servers to writing **infrastructure-as-code** (Terraform, Pulumi) and managing cloud spend (**FinOps**).

### Separation of Storage and Compute

The cloud's fast intra-datacenter networks enable a powerful pattern: **disaggregation**. Systems like Snowflake store data cheaply in S3 and spin up separate, ephemeral compute clusters only when a query runs. A 10 TB dataset costs pennies per month to store; you only pay for expensive CPUs the five minutes per month you actually need them.

---

## 6. Monoliths vs. Microservices

### The Monolith Problem

A monolithic application bundles every feature — auth, payments, inventory, email — into a single codebase and process. One memory leak in the image upload service brings down the checkout and homepage alike. Deploying a CSS fix requires coordinating every team and re-deploying the entire application.

### Microservices

Microservices decompose the monolith into small, independently deployable processes. The critical rule: **each service owns its own database**. Services communicate only through well-defined APIs (REST or gRPC). A failure in one service is contained — other services keep running.

The trade-off is substantial: you replace in-process function calls (nanoseconds, 100% reliable) with network calls (milliseconds, fail constantly). You also take on enormous operational complexity — distributed tracing, retry logic, service discovery, and network partitions.

**The rule of thumb:** microservices pay off when your organisation has enough engineers that the human coordination cost of sharing one codebase exceeds the engineering cost of managing the distributed system.

---

## 7. Serverless (Function-as-a-Service)

Serverless takes the pay-as-you-go model to its logical extreme. You deploy individual functions (AWS Lambda, Google Cloud Functions); the provider handles the infrastructure entirely. You pay only for the milliseconds your code actually executes.

| Advantage                                    | Trade-off                                              |
| -------------------------------------------- | ------------------------------------------------------ |
| Zero cost at zero traffic                    | **Cold start latency** (~100ms–1s on first invocation) |
| Scales to millions of requests automatically | **Stateless** — no memory between requests             |
| No OS patching or server management          | Requires event-driven, asynchronous architecture       |

{: .table .table-bordered .table-striped}

---

## 8. Data Privacy as an Architectural Constraint

The chapter closes with a dimension that many engineers treat as an afterthought: privacy law. GDPR (Europe), CCPA (California), and PCI-DSS (payment cards) impose hard engineering requirements.

The collision point is **append-only immutable logs** — a widely used pattern (Kafka, event sourcing) where deleting past records is either impossible or structurally corrupting.

Two approaches have emerged:

1. **Data minimisation** — only collect what is strictly needed. If you don't have the data, it can't be breached, leaked, or subpoenaed.
2. **Crypto-shredding** — encrypt each user's personal payload with a unique key. When they invoke the right to be forgotten, destroy the key. The bytes remain in the log but are permanently unreadable.

The deeper lesson: privacy laws are engineering constraints just like network latency or disk IOPS. Treating them as legal afterthoughts leads to architectures that literally cannot comply with the law.

---

## Key Takeaways

- There is no universally best architecture — every choice trades one set of costs for another.
- The OLTP/OLAP split exists because the physical access patterns of transactional and analytical workloads are fundamentally incompatible.
- Cloud elasticity shifts hardware from a capital cost to an operational cost and enables storage/compute disaggregation.
- Microservices solve a human coordination problem but create a distributed systems engineering problem.
- Privacy requirements must be designed in from day one, not retrofitted.

_Next: Chapter 2 — Defining Non-Functional Requirements (scalability, reliability, maintainability)._
