---
layout: post
title: "Part 6: Replication"
date: 2026-06-06
tags: ddia distributed-systems replication consistency databases
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

Saving a copy of data to a single machine means one hard drive failure can wipe everything out. _Replication_ — keeping the same data on multiple machines connected over a network — is the foundational technique that gives modern databases fault tolerance, geographic reach, and read scalability. Chapter 6 of DDIA reveals just how hard that simple-sounding idea actually is.

> ##### Source
>
> Notes drawn from Chapter 6 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. Why Replicate?

There are three distinct reasons engineering teams invest in replication:

| Goal                  | Description                                               |
| --------------------- | --------------------------------------------------------- |
| **High availability** | If one node dies, the others keep serving traffic         |
| **Reduced latency**   | Place data physically close to users in different regions |
| **Read scalability**  | Spread millions of concurrent reads across many replicas  |

The chapter organises the strategies for achieving these goals into three architectures: **single-leader**, **multi-leader**, and **leaderless** replication.

---

## 2. Single-Leader Replication

### The Hierarchy

One node is elected as the **leader** (also called master or primary). Every write request must be routed to it. The leader writes the change to its local storage, then streams a log of that change to **follower** nodes (replicas). Followers are read-only — they copy the leader's log and serve read traffic.

This architecture is the default for PostgreSQL, MySQL, and Kafka.

### Zero-Disk Architectures

The industry is shifting toward removing local hard drives entirely from database nodes. In a **zero-disk architecture (ZDA)**, all permanent data lives in object storage like Amazon S3. Nodes use RAM and fast local NVMe drives purely as ephemeral write buffers, asynchronously flushing batches to S3 in the background. This decouples compute from storage: a crashed node can be replaced by spinning up a fresh one and pointing it at the same S3 bucket.

### Synchronous vs Asynchronous Replication

This is one of the most consequential choices in the entire architecture.

**Synchronous:** The leader waits for the follower to confirm every write before acknowledging success to the client. If the follower is slow (garbage collection pause, dropped packet), the entire system halts. In practice, no production system runs fully synchronous because a single slow replica can bring writes to a standstill.

**Asynchronous:** The leader writes to its local write-ahead log (WAL), immediately acknowledges the client, and streams updates to followers in the background. If the leader dies before the background process syncs, any acknowledged-but-unsynced writes are permanently lost.

**Semi-synchronous (the real-world compromise):** Exactly one follower confirms synchronously, guaranteeing at least two durable copies. The rest catch up asynchronously.

---

## 3. Failover and Its Nightmares

When the leader fails, the system must:

1. **Detect the failure** — nodes exchange heartbeat messages; if no response arrives within a timeout (typically 30 s), the node is declared dead.
2. **Elect a new leader** — remaining nodes run a consensus algorithm (Raft, Paxos) and promote the follower with the most up-to-date log.
3. **Reconfigure routing** — all clients must be redirected to the new leader.

### Split-Brain

The most dangerous failure mode: the old leader was not dead, just temporarily disconnected. When it reconnects, it still believes it is the legitimate leader. Two nodes both accepting writes produces **split-brain** — two divergent truths that cannot be automatically merged in a single-leader system.

The industry's blunt solution is **STONITH** (_Shoot The Other Node In The Head_): a node detecting split-brain signals a power distribution unit to cut electricity to the old leader, physically preventing it from accepting further writes.

---

## 4. Eventual Consistency and Its Anomalies

Because followers replicate asynchronously, they may lag behind the leader by milliseconds or even minutes. During this window the system is **eventually consistent** — a weak guarantee that all replicas will converge _if_ writes stop, but offering no promise about intermediate states.

This lag creates three classical user-visible anomalies:

### 4.1 Read-After-Write (Read-Your-Writes)

A user writes data to the leader, then immediately reads it from a lagging follower. Their own update appears to have vanished.

**Fix:** Route a user's own reads back to the leader for a short window after a write. Other users' reads can safely go to followers because the reader has no expectation about the exact freshness of data they didn't write.

### 4.2 Monotonic Reads

A load balancer routes successive reads from the same user to different replicas. If the second replica is lagging more than the first, the user observes a comment appear, then disappear on the next page refresh — time appears to move backwards.

**Fix:** Hash the user ID to permanently pin each user to a specific replica for reads.

### 4.3 Consistent Prefix Reads

In a partitioned database, if a question and its answer are stored on different shards with uneven replication lag, an observer can see the answer before the question — causality is shattered.

**Fix:** Ensure causally related writes go to the same partition so the partition's internal log preserves their order.

---

## 5. Multi-Leader Replication

When users are distributed globally, forcing every write across an ocean to a single leader in one data center adds hundreds of milliseconds of latency. The solution: place a **leader in each major region**. A Tokyo write hits the Tokyo leader instantly; that leader then asynchronously forwards the change to the London leader in the background.

Multi-leader is also the model used by **collaborative offline apps** (notes apps, document editors): your phone's local SQLite database acts as a leader, syncing changes to the cloud when connectivity is restored.

### Leader Topologies

| Topology       | Structure                              | Risk                              |
| -------------- | -------------------------------------- | --------------------------------- |
| **Circular**   | A → B → C → A                          | One failed node breaks the chain  |
| **Star**       | One hub broadcasts to leaf nodes       | Hub is a bottleneck               |
| **All-to-all** | Every leader broadcasts to every other | Most resilient; ordering problems |

### Write Conflicts

Single-leader systems avoid conflicts entirely because writes are serialised through one node. Multi-leader systems cannot — two users can concurrently modify the same record on two different continents and both leaders accept the write.

**Conflict resolution strategies:**

| Strategy                            | Mechanism                                                                                    | Downside                                                                                              |
| ----------------------------------- | -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Operational Transformation (OT)** | Tracks the intent of every keystroke and transforms concurrent operations to preserve intent | Notoriously hard to implement correctly                                                               |
| **CRDTs**                           | Assigns a unique permanent ID to every element; operations commute in any order              | Deleted items leave invisible "tombstone" ghosts in memory, causing storage bloat                     |
| **Last Write Wins (LWW)**           | The write with the highest timestamp survives                                                | Clock skew between servers means the "winner" may actually be the older write; silently discards data |

---

## 6. Leaderless Replication

Leaderless systems (pioneered by Amazon Dynamo, used by Cassandra and Riak) eliminate the hierarchy entirely. Every replica can accept any write.

### Quorum Reads and Writes

Consistency is maintained through the **quorum condition**: with _N_ total replicas, if you write to _W_ nodes and read from _R_ nodes, and _W + R > N_, at least one node in every read set must overlap with every write set.

A typical configuration for _N_ = 5: _W_ = 3, _R_ = 3.

```
W + R = 3 + 3 = 6 > 5 = N  ✓
```

Even if two nodes are offline, any read of three nodes is mathematically guaranteed to include at least one node that witnessed the write.

### Read Repair

When a client reads from multiple replicas and sees different versions (identified by monotonically increasing version numbers), it writes the newest version back to the stale nodes. The act of reading heals the database.

### Version Vectors

A single version number only tracks absolute ordering from one node's perspective. A **version vector** is an array tracking each replica's version number independently — e.g. `{nodeA: 2, nodeB: 1, nodeC: 0}`. This creates a mathematical causal family tree of the data, allowing the system to distinguish:

- **Causal writes** (flour was added after milk → flour is a descendant)
- **Concurrent writes** (eggs and bacon added simultaneously → neither is an ancestor of the other)

### Concurrency Without Clocks

The most mind-bending insight in this section: two operations are **concurrent** if neither knows about the other — regardless of when they happened on a wall clock. If a network cable under the ocean is severed, a write made in London on Tuesday and a write made in Tokyo on Thursday are concurrent to the database, because neither server knew about the other's action when it was created. When the cable is repaired, those writes meet as peers with no causal relationship.

> Concurrency is formally defined by the _absence of a happens-before relationship_, not by physical time.

---

## 7. Architecture Comparison

| Approach          | Strengths                               | Weaknesses                                                           |
| ----------------- | --------------------------------------- | -------------------------------------------------------------------- |
| **Single-leader** | Simple ordering, no write conflicts     | Single point of failure, replication lag anomalies, complex failover |
| **Multi-leader**  | Low geographic latency, offline support | Mandatory conflict resolution, OT/CRDT complexity                    |
| **Leaderless**    | Extreme resilience, no failover needed  | No linear ordering, quorum math required, version vectors complex    |
