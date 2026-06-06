---
layout: post
title: "Part 7: Sharding"
date: 2026-06-06
tags: ddia distributed-systems sharding partitioning scalability databases
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

Chapter 6 showed how to keep _copies_ of data on multiple machines. Chapter 7 asks a different question: what do you do when your dataset is so large that no single machine can hold all of it? The answer is **sharding** — splitting the data into independent chunks and distributing those chunks across a fleet of nodes.

> ##### Source
>
> Notes drawn from Chapter 7 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. Vertical Scaling and Its Hard Limits

The instinctive first response to a slow database is to buy a bigger server — more RAM, more CPU cores, faster SSDs. This is **vertical scaling**, and it works well up to a point.

The wall it hits is the **NUMA (Non-Uniform Memory Access)** architecture of modern multi-socket servers. When a CPU needs data stored in the RAM bank of a different CPU socket, it must cross the motherboard's interconnect bus. As you add more sockets, cross-socket communication becomes a bottleneck. The machine spends increasing fractions of its time shuffling data internally rather than executing your queries.

On top of that, hardware pricing is super-linear: a server with 128 cores can cost five times more than a 64-core machine, not twice as much.

**Vertical scaling has a hard physical and economic ceiling.**

---

## 2. Horizontal Scaling: The Shared-Nothing Architecture

Instead of one enormous server, use a fleet of commodity machines. Each node handles a roughly equal fraction of the total data and query load. This is **horizontal scaling** (also called a _shared-nothing_ architecture).

A critical point: **sharding and replication are not mutually exclusive.** In production, shards are always replicated. A node might be the primary leader for Shard A while simultaneously acting as a follower replica for Shards B and C. If the primary for Shard A dies, the cluster promotes its replica with no data loss.

---

## 3. Multi-Tenancy and Isolation

Beyond raw throughput, sharding solves a second problem: **resource isolation between tenants** in SaaS products.

In a centralised database serving thousands of corporate clients, one client running a heavy analytical query can consume all available CPU and RAM — starving every other client. Assigning each tenant to their own shard creates hard resource boundaries.

Isolation benefits extend further:

- **Permission isolation:** A bug that drops a `WHERE` clause in application code cannot leak Tenant A's data to Tenant B when they are on physically separate machines.
- **Restore isolation:** Rolling back a specific tenant's shard to last hour's backup does not affect any other tenant on the platform.
- **Data privacy compliance (GDPR):** Deleting a European user's data is a localised, verifiable operation on their shard rather than a complex archaeological dig through shared B-trees.

---

## 4. The Enemy: Skew

The entire goal of a sharding strategy is an even distribution of data and query load. When the distribution is uneven, some nodes become **hotspots** while others sit idle. This is called **skew** — and it defeats the entire purpose of building a cluster.

---

## 5. Key-Range Sharding

Divide the total spectrum of primary keys into contiguous sorted ranges and assign each range to a node. Think of a printed encyclopedia: Volume 1 holds A–B, Volume 2 holds C–D, and so on.

**Advantages:**

- Data is stored in sorted order on disk (typically in SSTables or B-trees).
- **Range scans** are extremely fast: a query for "all temperature readings in July" locates the first entry for July 1 and streams sequentially to August 1 — no random seeks.

**The fatal flaw — write hotspots:**

If your partition key is a monotonically increasing timestamp and sensors write thousands of records per second, 100% of incoming writes land on the shard covering _right now_. Every other shard sits idle.

**Fix: compound keys.** Prepend the sensor ID to the timestamp — `sensor_73:2026-07-01`. Now writes for the current moment are spread across as many shards as there are sensors. The cost is that a query for "all sensors in July" must scatter across all shards and gather the results, increasing read complexity.

---

## 6. Hash Sharding

Pass the primary key through a hash function (e.g., Murmur3 or MD5) before routing. A good hash function has the **avalanche effect**: even slightly different input keys produce wildly different output values, uniformly scattered across the output space.

This **mathematically eliminates skew** from predictable input distributions.

**The cost:** all notion of ordering is destroyed. User `0001` and User `0002` will land on entirely different shards. Range queries become impossible.

---

## 7. The Modulo Trap

A natural implementation of hash sharding is to compute `hash(key) mod N`, where _N_ is the number of nodes. This routes each key to a node in range `[0, N-1]`.

**The problem:** when _N_ changes (a node is added or removed), almost every key maps to a different node. The cluster must physically migrate terabytes of data across the network — a migration storm that saturates bandwidth and can cause a full production outage.

---

## 8. Consistent Hashing with Fixed Logical Shards

The solution is to **decouple logical shards from physical nodes**.

Create a large fixed number of logical shards (e.g., 1,000 virtual buckets). The hash is always computed modulo 1,000 — a denominator that never changes. These 1,000 buckets are then assigned to physical nodes like dealing cards:

- 10 nodes → ~100 buckets per node
- Node fails → only its ~100 buckets are redistributed to surviving nodes; the other 900 are untouched.
- Node added → steal ~100 buckets from existing nodes; no wholesale rehashing.

This limits data movement to the absolute minimum required to maintain balance.

A variant used by Apache Cassandra is the **hash ring**: the hash output space is visualised as a circle, and each node "owns" a contiguous arc of that circle. Overloaded nodes split their arc and hand half to a newly spawned node.

---

## 9. Celebrity Hotspots: When Math Isn't Enough

Hash functions neutralise skew from _normally distributed_ key access. They are powerless against extreme concentration at a single key — the **celebrity problem**.

When a celebrity with half a billion followers posts a controversial photo, millions of concurrent requests target a single primary key (the celebrity's user ID). The hash function dutifully routes all of them to the same physical node, which melts under the load.

**Application-layer fix:** when the application detects a viral key, it appends a random 2-digit suffix (00–99) before writing. The comment for that post is now stored under 100 distinct keys, which the hash function scatters across 100 different nodes.

**The read trade-off:** to reassemble all comments, the application must issue a **scatter-gather** query to all 100 shards, wait for every response, merge the results in memory, sort them, and return them to the client. Write pressure is relieved at the cost of dramatically higher read complexity and latency.

---

## 10. Automated vs. Manual Rebalancing

Fully automated, aggressive rebalancing is dangerous. A temporary garbage collection pause can cause a node to miss a health-check heartbeat, triggering a false positive. The automated system declares the node dead and launches an emergency data migration. The migration itself saturates CPU and disk on the stressed node — causing it to actually crash. The traffic from the crashed node dumps onto healthy nodes, which now miss their heartbeats. The panic cascades. Within minutes the entire cluster is offline, brought down entirely by the algorithm that was trying to help.

**The safer approach:** use humans in the loop. An on-call engineer reviews a dashboard, recognises a temporary spike, and declines the rebalancing operation. Slower, but dramatically less likely to trigger a cascade failure.

---

## 11. Service Discovery and ZooKeeper

With data scattered across hundreds of dynamically changing nodes, how does a client application find the correct shard?

A dedicated **routing tier** (fleet of proxy servers) stands between clients and database nodes. Clients never talk to database nodes directly — they talk to the routing tier, which consults a **coordination service**.

**Apache ZooKeeper** is the classic example. It runs as a small cluster (3–5 nodes) using consensus algorithms to maintain a fault-tolerant, real-time map of which shard lives on which IP address.

Every database node holds an open TCP connection (heartbeat) to ZooKeeper and registers an **ephemeral node** in ZooKeeper's memory. If a database node dies, its TCP connection drops, ZooKeeper deletes its ephemeral node, orchestrates a replica promotion, and immediately pushes the updated map to the routing tier — all within milliseconds. Clients are transparently rerouted with no manual intervention.

---

## 12. Secondary Indexes in a Sharded Database

All of the above assumes you know the primary key. Real users search by secondary attributes ("red running shoes, size 10"). Secondary indexes on a sharded database force a hard architectural choice:

### Local Secondary Index (Document Partitioning)

Each shard maintains its own index covering only the data it holds.

| Operation | Performance                                                                             |
| --------- | --------------------------------------------------------------------------------------- |
| **Write** | Blazing fast — update the local index with no network coordination                      |
| **Read**  | Slow — must scatter-gather across every shard, suffering **tail latency amplification** |

**Tail latency amplification math:** if each node has a 1% chance of a slow response, querying 100 nodes gives a >60% chance of hitting at least one slow node. Your query is as fast as its slowest component.

### Global Secondary Index (Term Partitioning)

The index itself is sharded by the indexed term. The global index for "red" lives on a single dedicated node, which holds pointers to every red item across the entire cluster.

| Operation | Performance                                                                                                                                     |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Read**  | Fast — one targeted query to the index node returns all matching IDs                                                                            |
| **Write** | Slow — updating one record may require modifying index entries on several shards, requiring a **distributed transaction** (2PC) for consistency |

**The fundamental trade-off:**

- **Local index:** fast writes, slow scatter-gather reads
- **Global index:** fast targeted reads, slow distributed-transaction writes

The right choice depends on your application's read/write ratio.

---

## 13. Summary

| Concept                   | Core Idea                                                         |
| ------------------------- | ----------------------------------------------------------------- |
| **Vertical scaling**      | Hits NUMA and cost limits; eventually fails                       |
| **Key-range sharding**    | Sorted for range scans; beware write hotspots from monotonic keys |
| **Hash sharding**         | Even distribution; destroys ordering                              |
| **Consistent hashing**    | Fixed logical shards avoid migration storms on topology changes   |
| **Celebrity hotspot fix** | Append random suffix to viral key; pay with scatter-gather reads  |
| **ZooKeeper**             | Fault-tolerant real-time directory for shard-to-node mapping      |
| **Local index**           | Fast writes, slow scatter-gather reads                            |
| **Global index**          | Fast reads, slow distributed-transaction writes                   |
