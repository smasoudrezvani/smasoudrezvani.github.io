---
layout: post
title: "Part 8: Transactions"
date: 2026-06-06
tags: ddia distributed-systems transactions acid concurrency databases
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

At 2:00 a.m., a hospital system shows zero doctors on call. Each physician individually checked the rules and got approval to take sick leave. Nothing was coded incorrectly. The bug lived not in any single operation, but in the invisible gap _between_ operations. Chapter 8 of DDIA is about the mechanisms databases use to close that gap.

> ##### Source
>
> Notes drawn from Chapter 8 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. The Problem: Production Is Chaos

A database in a data center faces a relentlessly hostile environment:

- Hard drives develop bad sectors; SSDs wear out.
- RAM experiences bit flips from cosmic rays.
- Network cables get cut.
- Top-of-rack switches reboot mid-query due to firmware bugs.
- The OS out-of-memory manager kills your process mid-transaction.
- 10,000 concurrent clients race to write the same row simultaneously.

Without help from the database, an application developer must write thousands of lines of error-handling code to safely update a single record. **Transactions** exist to hide this chaos behind a clean abstraction.

---

## 2. The Transaction Abstraction

A **transaction** groups a sequence of reads and writes into a single logical unit with two possible outcomes:

- **Commit** — all operations succeed; all changes become visible and permanent.
- **Abort / Rollback** — something went wrong; the database undoes everything as if the transaction never happened.

The application developer doesn't need to handle partial failures manually. They simply open a transaction, do their work, and call commit. The database handles the rest.

### The NoSQL Detour

During the late-2000s NoSQL boom, transaction support was deliberately dropped to achieve web-scale throughput. The assumption was that strong guarantees and global scale were mutually exclusive. Developers were left to implement their own consistency checks in application code — a fragile, error-prone approach.

This assumption was disproven by **NewSQL** databases like Google Spanner, CockroachDB, and TiDB, which combine global sharding with consensus protocols to deliver strong transactional guarantees at massive scale.

---

## 3. ACID

The properties of transactions are captured by the acronym **ACID**, coined in 1983 by Haerder and Reuter. Despite its apparent precision, ACID is today partly a marketing term — different databases implement each letter differently.

### A — Atomicity

In ACID, atomicity is about **abortability**, not concurrency (despite the word's usage in multithreaded programming, where it means something entirely different).

If a transaction touches three tables and the server crashes after the second update, the database reverts the first two updates automatically.

**Mechanism — the undo log (write-ahead log):** Before modifying any real data file on disk, the database appends a note to an append-only log: _"I am about to change account A's balance from $500 to $400."_ If the system crashes, the recovery manager reads the log, sees no commit marker, and uses the recorded old values to reverse every change.

### C — Consistency

Consistency in ACID means the database is in a _good state_ according to your application's invariants — for example, in double-entry bookkeeping, debits and credits must always sum to zero.

**Critical point:** the database engine cannot enforce application-level business rules on its own. Foreign key constraints and uniqueness checks help, but defining the invariants and writing code that maintains them is the developer's responsibility. ACID merely promises that if your transaction preserves consistency, the database will safely store the result.

### I — Isolation

Multiple concurrent transactions should appear to run as if they are the only transactions on the system. This property is formally called **serializability**: the database guarantees that the final state of the data is identical to some valid serial (one-at-a-time) execution order.

Isolation prevents races like two users simultaneously reading a counter as 42 and both writing 43, losing one increment.

### D — Durability

Once the database acknowledges a commit, the data is safe even if the server immediately loses power.

In practice, durability is a **spectrum of risk reduction**, not an absolute guarantee:

- Writes are persisted to disk using `fsync` — but disk controllers lie, reporting success before data has left their own volatile cache.
- Data is replicated across multiple geographic regions.
- Regular backups are taken.

Even battle-tested databases have had subtle `fsync` bugs. Modern SSDs can silently corrupt data after extended power loss as electrons leak from floating gate transistors. Perfect durability does not exist; the goal is reducing the probability of loss to negligible levels.

---

## 4. Single-Object vs Multi-Object Operations

A single-row update within a transaction is simple. The real complexity emerges when a logical operation spans multiple tables or rows — for example, delivering an email requires:

1. Inserting the email body into the inbox table.
2. Incrementing the unread-message counter.
3. Adding an entry to the search index.

Without multi-object atomicity, a crash between steps 1 and 2 leaves the system permanently inconsistent: the email exists but the counter is wrong. Without isolation, a reader between steps 1 and 2 sees the email but a counter of zero — a coherent-looking but logically wrong state.

Pure key-value stores that prohibit multi-object transactions shift all this error handling to application code — a pattern that produces notoriously fragile systems.

---

## 5. Isolation Levels

Full serializability is expensive. Real databases offer a spectrum of **weak isolation levels** that trade safety guarantees for performance. Understanding which anomalies each level prevents is essential.

### Read Committed

The default isolation level in PostgreSQL, SQL Server, and many others. Provides two guarantees:

| Guarantee           | Meaning                                                                                                   |
| ------------------- | --------------------------------------------------------------------------------------------------------- |
| **No dirty reads**  | You only see data that has been fully committed — never a half-finished transaction's intermediate state  |
| **No dirty writes** | If two transactions try to write the same row simultaneously, one waits until the other commits or aborts |

**What it does not prevent — read skew (non-repeatable reads):** Aaliyah has $500 in account A and $500 in account B. A background job moves $100 from A to B. If Aaliyah reads account A ($500) before the transfer and account B ($600) after it, her app shows $1,100 — money appears from nowhere. Both reads returned fully committed data; the world just changed between them.

### Snapshot Isolation (Repeatable Read)

Designed specifically to eliminate read skew. The database gives each transaction a **consistent snapshot** of the data at the moment the transaction started. No committed changes made by other transactions after that point are visible.

**Implementation — MVCC (Multi-Version Concurrency Control):**

PostgreSQL does not overwrite rows in place. An `UPDATE` is implemented as a logical delete of the old row and an insert of a new row. Both rows coexist on disk with hidden metadata:

- `xmin` — the transaction ID that inserted this version of the row.
- `xmax` — the transaction ID that deleted it (or null if still current).

If your transaction has ID 12, and a row was modified by transaction 13, the database's visibility rules hide the modification from you (13 > 12, so it happened "in the future" relative to your snapshot). You see the pre-modification row as if it were still current.

**The critical property of MVCC: readers never block writers, and writers never block readers.** A long-running analytics query or backup can scan the entire database without holding locks, while concurrent writes proceed unimpeded.

A background **vacuum daemon** periodically cleans up old row versions once it is certain no active transaction needs them.

---

## 6. Write-Skew and Phantoms

Snapshot isolation eliminates read anomalies but does not protect against all write conflicts.

### Write-Skew

The hospital scenario from the introduction is **write-skew**: two transactions each read a shared condition, each decide it is safe to proceed, and each write to _different_ rows — yet their combined effect violates a business invariant.

The hospital invariant: at least one doctor must be on-call.

```
T1 (Dr. Aaliyah)          T2 (Dr. Bryce)
SELECT COUNT(*)           SELECT COUNT(*)
→ 2                       → 2 (same snapshot)
(2 ≥ 1, safe to leave)   (2 ≥ 1, safe to leave)
UPDATE → off-call         UPDATE → off-call
COMMIT                    COMMIT
→ 0 doctors on-call  ✗
```

Neither transaction overwrote the other's row. A `SELECT FOR UPDATE` on Aaliyah's row would not help Bryce, because he is updating his own row.

### Phantoms

A phantom occurs when a write in one transaction changes the result of a _search query_ in another. If you search for overlapping meeting room bookings before inserting your own, a concurrent insert from another transaction creates a new row that your search did not see — a "phantom" row that should have stopped your booking.

**Materialising conflicts (last resort):** Pre-populate a table with rows representing every possible time slot for every room, so the application has a concrete row to `SELECT FOR UPDATE` before checking availability. This works but pollutes the data model with artificial rows.

---

## 7. Serializable Isolation: Three Approaches

Full serializability — the guarantee that the outcome is identical to some serial execution — is achievable three ways, each with distinct trade-offs.

### 7.1 Actual Serial Execution

Literally run one transaction at a time on a single CPU core, using stored procedures so there is no round-trip latency. Used by Redis and VoltDB.

**Works when:** all active data fits in RAM and transactions are tiny and fast.

**Bottleneck:** scaling beyond one core requires partitioning, and cross-partition transactions stall while coordinating over the network.

### 7.2 Two-Phase Locking (2PL)

The classical approach, used for decades. Writers block readers; readers block writers.

- To **read** a row: acquire a _shared lock_. Many can hold it simultaneously.
- To **write** a row: acquire an _exclusive lock_. No one else can read or write while you hold it.

2PL fully prevents write-skew: each transaction locks the rows it reads, so another transaction cannot modify those rows mid-execution.

**Cost:** heavy lock contention, reduced throughput, and **deadlocks** (Transaction A holds row 1 and waits for row 2; Transaction B holds row 2 and waits for row 1). The database must run background deadlock detection and forcefully abort one side, requiring a retry.

### 7.3 Serializable Snapshot Isolation (SSI)

The modern approach, implemented in PostgreSQL and CockroachDB. It combines the MVCC snapshots of snapshot isolation with a monitoring layer that detects conflicts.

**Key insight:** instead of preventing conflicts via blocking locks, assume conflicts are rare and let transactions run freely. Track what each transaction has read, and abort only when a commit would produce a non-serializable outcome.

Applied to the hospital scenario:

1. Dr. Aaliyah's transaction commits first. She is now off-call.
2. Dr. Bryce's transaction attempts to commit.
3. SSI checks: "Did Bryce's read of the on-call count become stale because another transaction modified it?" Yes — Aaliyah changed it.
4. SSI aborts Bryce's transaction. He retries, now sees count = 1, and his leave request is correctly rejected.

**Result:** readers don't block writers; writers don't block readers; full serializability is maintained. The overhead only appears on the rare transaction that actually conflicts.

---

## 8. Distributed Transactions

All of the above describes single-node databases. When a transaction must span multiple shards or services — for example, deducting $100 from a New York shard and crediting $100 to a Tokyo shard — we need distributed atomicity.

### Two-Phase Commit (2PC)

A **coordinator** node orchestrates a two-phase protocol:

**Phase 1 — Prepare:** Coordinator asks all participants: "Can you commit?" Each participant checks constraints, writes its undo log to disk, and replies yes or no. A yes is an irrevocable promise — the participant surrenders its right to abort unilaterally.

**Phase 2 — Commit:** If all say yes, coordinator broadcasts commit. If anyone said no, coordinator broadcasts abort.

**The fatal flaw:** if the coordinator crashes between Phase 1 and Phase 2, participants are left **in doubt** — they cannot abort (they promised) and they cannot commit (they haven't received the order). They hold locks on database rows indefinitely. The entire system stalls until the coordinator recovers.

**XA transactions** extend 2PC across heterogeneous systems (e.g., PostgreSQL + Apache Kafka). The standardised lowest-common-denominator protocol performs poorly and cannot detect cross-system deadlocks.

### Consensus-Backed Distributed Transactions

NewSQL databases replace the single coordinator with a **consensus cluster** (e.g., Raft or Paxos). The coordinator's state — the transaction intent log — is replicated across 3–5 nodes. A quorum must acknowledge each log entry before it is committed.

If the leader coordinator dies, the surviving nodes detect the failure via missed heartbeats, elect a new leader in milliseconds, and resume the transaction from where the replicated log left off. There is no stuck _in-doubt_ state because the log is durable across the cluster.

This is the architectural foundation of Google Spanner and CockroachDB: strong, distributed ACID transactions that survive coordinator failures without stalling.

---

## 9. Summary

| Concept                       | Core Idea                                                                                                  |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Atomicity**                 | All-or-nothing via undo log; abortability is the key property                                              |
| **Consistency**               | Application's responsibility; database enforces constraints, not business rules                            |
| **Durability**                | Spectrum of risk reduction; `fsync`, replication, backups                                                  |
| **Isolation**                 | Concurrency safety; serializability is the gold standard                                                   |
| **Read Committed**            | Prevents dirty reads and dirty writes; allows read skew                                                    |
| **Snapshot Isolation / MVCC** | Consistent snapshot via xmin/xmax; readers never block writers                                             |
| **Write-Skew**                | Concurrent transactions each check a shared condition, then write to different rows, breaking an invariant |
| **SSI**                       | Optimistic serializability; abort only on actual conflict                                                  |
| **2PC**                       | Distributed atomicity; coordinator is a single point of failure                                            |
| **Raft/Paxos in NewSQL**      | Replicated coordinator eliminates the 2PC crash risk                                                       |
