---
layout: post
title: "Part 10: Consistency and Consensus"
date: 2026-06-15
tags: ddia distributed-systems consistency consensus linearizability raft zookeeper
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

On a single machine, the illusion is perfect: you write a value, you read it back, it's there. But the moment you replicate that value across servers in different cities, you have to answer a deceptively hard question — which copy is true? Chapter 10 of DDIA is about the mechanisms that restore that illusion, the trade-offs they force, and the mathematical bedrock underneath it all.

> ##### Source
>
> Notes drawn from Chapter 10 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. Two Philosophies: Eventual vs. Strong Consistency

Whenever you replicate data across multiple nodes for safety or speed, the copies can get out of sync. How you handle that divergence defines the character of your system.

### Eventual Consistency

Copies may briefly disagree, but will converge — _eventually_. The upside is speed and availability: writes succeed immediately on the local node without waiting for remote confirmation. The downside is that conflict resolution lives in your application code. Double-bookings, stale reads, and race conditions become your problem to handle with read repairs, anti-entropy processes, and apology emails.

This is the default philosophy of multi-leader and leaderless databases like Cassandra and DynamoDB — optimized for throughput, not strict correctness.

### Strong Consistency

Before confirming a write to the user, the database silently coordinates with remote nodes to lock the record. Only once every relevant node agrees does the system return "success." From the application developer's perspective, the multi-node cluster behaves exactly like a single machine — no conflicts, no stale reads, no double-bookings.

The cost: every write is limited by network latency, and a severed network link can take the system offline entirely even when the servers themselves are healthy.

**Which should you choose?**

| Domain                                               | Recommendation                                  |
| ---------------------------------------------------- | ----------------------------------------------- |
| Social media likes, non-critical feeds               | Eventual — delayed visibility hurts no one      |
| Financial ledgers, medical records, unique usernames | Strong — silent data corruption is unacceptable |

---

## 2. Linearizability: The Rule Book Behind the Illusion

Linearizability is the strict mathematical definition of strong consistency. It provides a single guarantee:

> **Once a write is observed by any client, no subsequent read by any client may return an older value.**

It enforces the arrow of time. Once the system reveals a new truth, it can never backslide — regardless of which server you talk to.

### When the Illusion Breaks

Imagine two friends, Aaliyah and Bryce, in the same room watching a football final. The game ends. Aaliyah refreshes her phone and reads the final score from the primary database. She shouts the result. Two seconds later, Bryce refreshes the same app. But Bryce's request routes to a read replica that is lagging by a few seconds — it still shows the game in progress.

Bryce's later read returned older data than Aaliyah's earlier read. That is a linearizability violation. The system allowed the clock to run backwards.

### The Gray-Area Window

A write takes time to process — say 50 ms. Reads that arrive during that window are in an ambiguous state: they might hit the database before the disk write, returning the old value, or after, returning the new one. Linearizability tolerates this gray area during the write. But the moment any client successfully reads the new value, the write has been _observed_. From that point on, every subsequent read everywhere must also return the new value. The line has been crossed; there is no going back.

---

## 3. Linearizability vs. Serializability

These terms are constantly confused. They are entirely different properties.

| Property            | Scope                                        | What it guarantees                                                                                    |
| ------------------- | -------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Serializability** | Transactions (multiple reads/writes grouped) | Running N concurrent transactions produces the same result as some serial order of those transactions |
| **Linearizability** | Single objects (one row, one register)       | Reads always return the most recently written value                                                   |

A database can be serializable but not linearizable — transactions are safely isolated from each other, but they might be operating on data that is several minutes out of date.

**Right skew** is the classic serializability bug: a hospital system requires at least one doctor on call at all times. Both doctors check simultaneously, see two doctors on call, and both click "go off call." Without serializability, both transactions pass the check and commit. Zero doctors remain. Serializability prevents this by isolating the transactions. But it says nothing about whether the roster data is fresh.

---

## 4. Where Linearizability Is Non-Negotiable

### Leader Election and Distributed Locks

If five database nodes all crash and reboot simultaneously, they all try to become the leader. If the locking mechanism is not linearizable, two nodes may both believe they successfully acquired the lock at the same moment — a split-brain. You now have two primaries accepting conflicting writes from users, two divergent datasets, and no safe way to merge them.

A linearizable lock acts like a ticket dispenser: even if two nodes press the button at the exact same microsecond, exactly one receives ticket 1 (the lock) and the other immediately receives "out of stock."

### Cross-Channel Race Conditions

A user uploads a photo via a web server. The server saves the file to object storage (slow) and simultaneously sends a message through a queue (fast) telling a background worker to resize it. If the storage system isn't linearizable, the worker may arrive at storage before the file has finished replicating internally, receiving a "file not found" error for a file that was just written.

The storage system must guarantee: once the web server is told the file is written, any subsequent read is 100% guaranteed to see it.

### Why Quorums Don't Save You

A common misconception: "if I require a majority quorum for both reads and writes, I get linearizability." You do not.

Imagine a write propagating to five nodes. It reaches nodes A and B but is delayed on C, D, and E due to network congestion. Client 1 does a quorum read, hits A, B, C — sees the new value on A and B, returns it. Linearizability now demands all subsequent reads return the new value. But Client 2 immediately does a quorum read, hits C, D, E — none of which have the new value yet. Client 2 travels back in time.

Quorum read/write sets overlap, but they don't resolve synchronously across all nodes. Quorums do not guarantee linearizability without additional consensus protocols layered on top.

---

## 5. The CAP Theorem — and Its Limits

The CAP theorem forces a choice during a network partition:

- **CP (Consistency + Partition Tolerance)**: Halt all operations until the partition heals. You stay correct but go offline. Users get an error; no data is corrupted.
- **AP (Availability + Partition Tolerance)**: Continue serving requests on both sides of the partition. You stay online but sacrifice linearizability. Risk of double-bookings, stale reads.

The theorem is a useful mental model for beginners but is considered too simplistic for real engineering:

1. **Partition tolerance isn't a choice.** You cannot decide that cables will not be cut by falling trees. Partitions happen. So you only ever choose between C and A _when a partition occurs_.
2. **It ignores degraded performance.** Real outages rarely look like a clean binary partition. Usually you get a network that is simply slow, dropping 10% of packets, or a node frozen by GC. Is that a partition or high latency? The system can't tell.
3. **We abandon linearizability for performance, not just during failures.** Even a multi-core CPU drops linearizability: one core writes to its L1 cache and only asynchronously flushes to RAM. Another core reading the same memory address may see the old value. We sacrifice strict recency inside a silicon chip to keep CPU execution fast. Across a 500 km fiber link, the trade-off is even more unavoidable.

---

## 6. Ordering Without Clocks: Lamport Timestamps

We need to order events across nodes without relying on physical clocks (which drift, lie, and jump backwards on leap seconds). The solution is a **logical clock**.

### The Algorithm

Every node keeps a simple integer counter. Two rules:

1. Before sending any message, increment your counter.
2. On receiving a message, if the message's counter is higher than yours, fast-forward your counter to match it, then add 1.

This creates a mathematical chain of causality. If node A reads a message from node B and then sends a new message, A's new counter is guaranteed to be strictly higher than B's. The Lamport timestamp proves that A _knew about_ B's event.

### Total Ordering with Tiebreakers

What if two nodes — having never communicated — both generate counter value 3 at the same microsecond? Lamport timestamps break ties by appending the node ID. Counter 3 from node A becomes `3-A`; counter 3 from node B becomes `3-B`. The rule is deterministic and consistent across the whole cluster. It doesn't matter who physically wrote first — only that every node applies the same tiebreaker.

### The Critical Limitation: Out-of-Band Causality

Lamport clocks only know about messages they have explicitly seen. They are blind to causality that bypasses the message chain.

**The privacy breach scenario:**

1. A user opens their laptop and sets their account to **private**. The accounts database tags this write with logical timestamp **15**.
2. Immediately after, the user picks up their phone and uploads a personal photo. This request routes to the photos database, which has a slightly lagging counter. It assigns the photo upload logical timestamp **12**.
3. A viewer loads the page. The system reasons: photo was uploaded at time 12; account went private at time 15; therefore at time 12 the account was still public — show the photo.
4. A private photo is leaked to the public internet because of a lagging integer.

The user communicated causality out-of-band (laptop then phone). The logical clocks had no way to track it.

**Mitigations:**

- **Hybrid Logical Clocks (HLC)**: Combine physical time (NTP-synced) with Lamport counters. Keeps nodes in roughly the same physical time zone while preserving causality for events in the same millisecond.
- **Vector Clocks**: Each node keeps an array of counters, one per node. Expensive — a 100-node cluster carries 100 integers on every message — but allows the system to definitively identify _concurrent_ events (where neither node knew about the other), rather than just arbitrarily ordering them.

---

## 7. Consensus: Agreement Under Chaos

Logical clocks improve ordering but cannot prevent race conditions that require the whole system to stop and agree on a single truth. For that, we need **consensus**: getting a group of independent nodes to agree on one value and never change their minds about it.

### The FLP Impossibility Result

In 1985, Fischer, Lynch, and Paterson proved mathematically that in a completely asynchronous system — no clocks, no timeouts, network delays potentially infinite — it is impossible to guarantee that a consensus algorithm will always terminate if even one node might crash.

Why? Without a timeout, you can never distinguish a crashed node from a very slow one. You wait forever. You can never safely declare it dead and move on without risking a split-brain.

**How Raft and Paxos get around it**: They cheat. In the real world, we use physical clocks and timeouts aggressively. If a node doesn't respond in 150 ms, we declare it dead, remove it from the quorum, and proceed. The FLP result relies on the strict academic assumption of no timeouts — an assumption that never holds in practice.

### Epochs and Leader Election in Raft

Consensus algorithms don't elect a leader forever — they elect temporary leaders for a numbered **epoch** (Raft calls it a _term_). This prevents split-brain when an old leader disconnects and reconnects:

1. Node 1 is the leader for epoch 4, sending heartbeat pings every 50 ms.
2. Node 1's network cable is unplugged. Heartbeats stop.
3. Node 2's election timeout fires (say 150 ms). It increments the epoch to **5**, declares itself a candidate, and requests votes.
4. A node grants its vote to the first candidate it hears from in epoch 5, provided that candidate's data is at least as up-to-date as its own.
5. If Node 2 collects a majority (3 of 5), it becomes leader for epoch 5 and starts sending heartbeats.
6. Node 1's cable is plugged back in. It tries to issue a command tagged with epoch 4. Every other node replies: "We are on epoch 5. Your authority is rejected." Node 1 steps down and becomes a follower.

**Split election prevention**: If two nodes timeout simultaneously and both run for leader, they split the votes. Raft solves this with randomized election timeouts — Node 2 might wait 150 ms while Node 3 waits 165 ms. The small random offset almost always ensures one node starts its campaign before the other.

---

## 8. The Unified Field Theory of Distributed Systems

One of the most profound insights in this chapter: **single-value consensus, distributed locks, shared logs, and atomic commit are all mathematically equivalent**.

| Operation                  | What it actually requires                                                                    |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| **Compare-and-set lock**   | All nodes agree on who successfully changed a variable first — single-value consensus        |
| **Append to a shared log** | All nodes agree on which entry occupies position N — single-value consensus on the next slot |
| **Atomic commit (2PC)**    | All nodes agree on a single binary value: commit or abort — single-value consensus           |

If you solve single-value consensus, you can build a distributed lock. With a reliable lock, you can build a distributed log by locking the next empty slot. With a reliable log, you can implement atomic commit by writing "COMMIT" to the log. Solve one, solve them all.

---

## 9. ZooKeeper and etcd: Outsourcing the Hard Problem

Implementing Raft or Paxos correctly is notoriously difficult. A microscopic bug in timeout handling will silently corrupt your entire database. In practice, developers outsource consensus to specialized coordination services.

### Why Not Run Consensus Across All Nodes?

In a 10,000-node database cluster, running a consensus vote for every write would be catastrophic. Consensus requires a majority of nodes to exchange packets before every decision. Network traffic scales quadratically. Latency climbs from milliseconds to minutes. The system chokes on its own voting overhead.

### The Pattern: A Tiny Consensus Cluster

Instead:

1. Deploy a **dedicated cluster of 3–5 nodes** running ZooKeeper or etcd. These nodes do nothing but run the consensus algorithm.
2. All 10,000 database nodes act as **clients** of that small cluster.
3. When the database cluster needs a leader election, a distributed lock, or a configuration value, it asks the 3–5 node ZooKeeper cluster to decide. ZooKeeper runs fast consensus on 5 nodes, hands back the result.

Think of it as a tiny Supreme Court that the rest of the system consults for undeniable, final decisions.

### What Coordination Services Provide

- **Distributed locks**: Only one worker node processes a critical job at a time.
- **Leases**: Locks with automatic expiry — if the holder dies, the lock releases without any cleanup code.
- **Fencing tokens**: Sequentially increasing integers handed out with each lease, used to reject zombie writes from nodes with expired leases (see Part 9).
- **Configuration management**: Update a config value once in ZooKeeper; it safely propagates to all 10,000 nodes. No SSH loops.
- **Failure detection via ephemeral nodes**: Every active node maintains a live heartbeat session with ZooKeeper and creates an _ephemeral_ record saying "I am alive." When the heartbeats stop, ZooKeeper's timeout fires, automatically deletes the ephemeral record, and pushes an instant notification to every watcher: "Node 7 is dead." The cluster triggers failover without any polling.

This exact pattern is the backbone of Apache Kafka, Kubernetes, and Apache Spark.

---

## Summary

| Concept                                  | The problem                                         | The solution                                                                    |
| ---------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Eventual consistency**                 | Copies diverge; conflicts are application's problem | High availability and throughput; accept occasional stale reads                 |
| **Strong consistency / Linearizability** | Strict recency guarantee across all nodes           | Coordinate before confirming writes; slower, lower availability                 |
| **CAP theorem**                          | Network partitions force a choice                   | Pick C (halt) or A (serve stale data); CAP is a useful but oversimplified model |
| **Lamport timestamps**                   | Physical clocks drift and lie                       | Logical counters track causality without wall clocks                            |
| **Vector clocks**                        | Lamport can't detect concurrent events              | Per-node counter arrays; expensive but expressive                               |
| **FLP impossibility**                    | Consensus cannot terminate in a pure async system   | Use timeouts to declare nodes dead and move on                                  |
| **Raft / Paxos**                         | Fault-tolerant leader election and agreement        | Epochs, quorums, randomized timeouts                                            |
| **ZooKeeper / etcd**                     | Implementing consensus correctly is too hard        | Outsource to a dedicated 3–5 node coordination cluster                          |

The modern cloud is a stack of imperfect hardware and unreliable networks held together by linearizability guarantees, logical clocks, consensus algorithms, and the principle that a majority agreement — even among confused, slow, and occasionally dead nodes — is enough to construct a reliable truth.
