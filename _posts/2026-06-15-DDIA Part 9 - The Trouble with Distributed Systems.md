---
layout: post
title: "Part 9: The Trouble with Distributed Systems"
date: 2026-06-15
tags: ddia distributed-systems networks clocks fault-tolerance consensus
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

On a single machine, a crash is clean. The OS falls on its sword, points at the memory address, and stops. We prefer a dead computer to one that silently calculates the wrong payroll. But the moment you spread your system across 50 machines talking over a network, that comforting determinism evaporates. Chapter 9 of DDIA is about why — and how to build reliable systems out of fundamentally unreliable parts.

> ##### Source
>
> Notes drawn from Chapter 9 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. Partial Failure: The Defining Property

On a single computer, failure is binary: working or crashed. In a distributed cluster of 100 nodes, **partial failure** is the norm. Parts of the system break in unpredictable, non-deterministic ways while other parts keep running:

- A network link between two racks degrades and starts dropping 10% of packets.
- Node 42's hard drive begins failing, making reads 5,000× slower — but the node hasn't crashed.
- A write command sometimes succeeds, sometimes fails, and sometimes disappears with no response at all.

The binary working/crashed model is gone. Dealing with that uncertainty requires a completely different architectural approach.

### Why We Embrace Failure Instead of Preventing It

The naive response is to build an infallible system. The industry learned to do the opposite: **assume that components will fail** and design the system to handle partial failures gracefully.

The payoff is a massive operational superpower: if the system can survive individual nodes going offline unexpectedly, you can take them offline **intentionally**. Rolling upgrades become trivial — take down node 1, patch it, bring it back. The load balancer shifts traffic to nodes 2 and 3. Users see nothing. A huge infrastructure upgrade happens invisibly beneath their feet.

---

## 2. The Network Lies

In a shared-nothing architecture, machines communicate exclusively by passing messages over the network. And the network is worse than unreliable — it **actively masks truth**.

### Three Failure Scenarios You Cannot Distinguish

When a client sends a request and gets no response, there are (at least) three possible explanations:

1. **Request lost in transit** — a fiber cable was cut by a backhoe.
2. **Server crashed or unresponsive** — the node never saw the message.
3. **Server processed it successfully, but the response was lost** — the database wrote your credit card charge and dropped the receipt.

From the client's perspective, all three look identical: silence. The client has no way to know which scenario happened. Did the charge go through or not?

### Timeouts: The Only Defense

The only mechanism we have to detect faults in an asynchronous network is the **timeout**: after waiting N seconds with no acknowledgment, declare failure and move on.

But choosing the right timeout value is not science — it's a statistical guess based on your network's historical latency percentiles:

- **Too low**: false positives. You declare a node dead when it's just slow. Retries add traffic to an already-congested network.
- **Too high**: agonizingly slow recovery. Users stare at a spinner for a full minute before seeing an error.

---

## 3. TCP, UDP, and the Cost of Reliability

### TCP and Backpressure

If a powerful sender transmits data faster than the receiver can process it, the receiver's memory buffers overflow and it silently drops packets. TCP solves this with a **sliding window** mechanism:

- The receiver advertises how much buffer space it has available.
- As the sender transmits, that window shrinks.
- When the window hits zero, the sender is forbidden from sending more until the receiver acknowledges and reopens the window.

This prevents receiver crashes from overwhelming traffic — but it comes at a cost. If a packet is lost, TCP withholds all subsequent packets in a buffer, silently retransmits the missing one, reassembles everything in order, and then hands the complete sequence to the application. To your software, this looks like the network froze for half a second.

### UDP: Fire and Forget

UDP throws reliability out entirely. No connections, no backpressure, no retransmission. It fires datagrams into the void as fast as the network interface allows.

For latency-sensitive applications — VoIP, video conferencing — this is the right call. If a packet carrying a syllable of your voice gets lost, you don't want TCP to pause the entire audio stream for 300ms to fetch the missing syllable. By the time it arrives, the conversation has moved on. **Delayed audio data is worthless data.** Drop the packet, glitch for a fraction of a second, and keep the real-time stream moving.

### Queuing Delay and the Funnel

Inside a data center switch, multiple input ports try to route packets to the same output port simultaneously. Since two packets cannot occupy the same wire at the same time, the switch queues them in a memory buffer. If input traffic floods in at 50 Gbps but the output port drains at 10 Gbps, that buffer fills fast. When it's full, the switch **drops packets**.

This queuing delay is the primary source of unpredictable latency in modern networks — not broken cables.

### Why We Chose This Architecture

The alternative — dedicated private circuits between every pair of nodes — eliminates queuing but wastes 99% of capacity. The old analog telephone network worked this way (circuit switching): a physical copper path was clamped together for the duration of every call. Unused bandwidth during silences was wasted.

Modern data centers use **packet switching**: shared wires, chopped-up data, best-effort delivery. Variable delays are not a law of physics. They are the deliberate economic choice to maximize utilization over predictability. We traded latency guarantees for cheap, high-throughput bandwidth.

---

## 4. Clocks Are Lying Too

Timeouts require knowing how much time has elapsed. But a computer's measurement of time is deeply flawed.

### Quartz Drift

Inside every server is a quartz crystal oscillator. When current passes through it, it vibrates at a specific frequency and a counter translates those vibrations into seconds. But quartz drifts — and the rate of drift changes with **temperature**. If a server rack goes from 20°C to 26°C because the air conditioning fails, the quartz crystal physically expands, its vibration frequency changes, and the clock runs faster or slower.

### NTP's Fundamental Flaw

NTP daemons compensate by querying atomic-clock-backed time servers over the network. But NTP assumes the network journey to the server takes exactly half the time of the round trip. If routing is asymmetric — which it almost always is — NTP calculates the correction incorrectly. Even with NTP running perfectly, a server's clock can be off by tens or hundreds of milliseconds.

### The Leap Second Nightmare

Earth's rotation is gradually slowing. Occasionally the international body that governs time inserts a **leap second**: a specific minute contains 61 seconds. Most software was written with the hard assumption that a minute always has exactly 60 seconds. When an NTP server tells your Linux machine "it is still 23:59:59" for two consecutive seconds, the kernel's high-resolution timers can enter an infinite loop. In 2012, a leap second caused a Linux kernel bug that spiked CPU to 100% and took Reddit, Mozilla, and several airlines completely offline.

### Monotonic Clocks: Stopwatches, Not Calendars

Operating systems provide two types of clocks:

| Clock type      | Jumps backward?              | NTP-adjusted? | Use for                     |
| --------------- | ---------------------------- | ------------- | --------------------------- |
| **Time-of-day** | Yes (NTP, leap seconds, DST) | Yes           | Calendar timestamps         |
| **Monotonic**   | Never                        | No            | Measuring elapsed durations |

For timeouts, always use the **monotonic clock**. Record the counter before sending a request, then compare when checking for expiry. The absolute value is meaningless (it might be milliseconds since last reboot), but the _difference_ is reliable.

The fatal limitation: you cannot compare monotonic clocks **across machines**. Node A's counter of 500 and Node B's counter of 5,000 tell you nothing about which event happened first. For ordering distributed events, you must use time-of-day clocks — and accept their unreliability.

### Last Write Wins and Silent Data Loss

Many databases use **Last Write Wins (LWW)**: when two nodes have conflicting data, keep the write with the newest timestamp. The problem:

1. Client A writes `x = 1` to node 1. Node 1 tags it with timestamp `100`.
2. Client A's write replicates to node 2.
3. Client B reads `x = 1` from node 2, then writes `x = 2`. Node 2's clock is drifting slightly behind — it tags the write with timestamp `99`.
4. A third node receives both writes, runs LWW, and keeps `x = 1` (timestamp 100). Client B's increment is silently discarded.

Client B read the state, made a logically valid update, and the database threw it away because of a few milliseconds of hardware clock drift. In a financial system, money just vanished.

### Google Spanner's TrueTime

Google's solution: stop pretending clocks are synchronized. Install GPS receivers and atomic clocks directly in data center racks to minimize error — then **quantify the remaining uncertainty**.

Instead of returning a timestamp, the **TrueTime API** returns a confidence interval: "I cannot tell you exactly what time it is, but the true time is somewhere between 2:14:59 and 2:15:01." When a transaction wants to commit, Spanner reads the interval's upper bound and **deliberately waits** for that entire window to pass before committing. This guarantees that any subsequent transaction will have a strictly later interval with no overlap — causal ordering without relying on quartz crystals.

---

## 5. Process Pauses and the Zombie Problem

Even CPU execution is not the continuous flow it appears to be. An entire application — every thread, every calculation — can be **frozen for seconds or minutes** without knowing it happened.

### Sources of Process Pauses

- **Stop-the-world garbage collection**: JVM, CLR, and Go runtimes periodically halt every application thread to reclaim memory. Large Java heaps can pause for tens of seconds.
- **Synchronous disk I/O**: If a file isn't in the RAM cache, the calling thread blocks until the disk seeks.
- **Memory paging**: When the OS swaps memory to disk, a thread accessing a paged-out variable is frozen until the OS reads it back.
- **VM live migration**: Cloud providers can pause your entire virtual machine, copy its RAM over the data center network to a different physical rack, then resume execution. Multi-second blackout; application never knew.

### The Zombie Scenario

Here's why pauses are dangerous — they don't just slow the system down, they create **zombie leaders**:

1. Node A requests a lease (a distributed lock with a timeout): "I want exclusive write rights for 10 seconds."
2. Node A gets the lease. One second in, a massive GC pause freezes it.
3. Real wall-clock time ticks on. The 10 seconds elapse. The cluster notices Node A's heartbeats have stopped and elects Node B as the new leader.
4. Node B begins writing to the database.
5. GC finishes. Node A wakes up **mid-execution**, doesn't bother to check the clock, and confidently sends a write — because its internal state still says "I hold the lease."

Node A is a zombie: walking around with authority that mathematically expired, overwriting Node B's valid data.

---

## 6. Fencing Tokens: Killing Zombies

The fix is to make the **storage layer** actively reject stale authority — not rely on the zombie to check its own lease.

Every time a lease is granted, the lock service hands out a **sequentially increasing fencing token**:

- Client A gets lease + token **33**.
- Client A pauses. Lease expires. Client B gets lease + token **34**.
- Client B writes to storage with token 34. Storage records: "highest token seen = 34."
- Zombie Client A wakes up and tries to write with token 33.
- Storage compares: 33 < 34. **Write rejected.** Zombie fenced off.

No clock comparisons. No timeouts. Just a monotonically increasing integer that the storage layer checks on every write.

### Fencing Tokens in Leaderless Databases

When there's no single central storage gatekeeper, fencing tokens can be embedded directly into write timestamps. Token 34 generates timestamps starting with `34...`; token 33 generates timestamps starting with `33...`. Any string starting with `34` is lexicographically larger than one starting with `33`, so LWW always picks the legitimate leader — regardless of quartz drift.

---

## 7. Byzantine Faults

Everything above assumes nodes are **honest**: slow, crashed, or confused — but not malicious. A zombie node that overwrites data does so because it doesn't know its lease expired, not because it's trying to subvert the system.

A **Byzantine fault** is when a node actively lies: a compromised node crafts a fake fencing token of `999,999` to bypass the storage guard, or sends contradictory messages to different peers to prevent consensus.

### When Byzantine Tolerance Matters

| Environment                           | Assume Byzantine?                                                        |
| ------------------------------------- | ------------------------------------------------------------------------ |
| Private enterprise data center        | No — you own the hardware; garbage is almost always a bug, not an attack |
| Public blockchain (Bitcoin, Ethereum) | Yes — anonymous, mutually untrusting participants; assume active fraud   |

Building Byzantine fault tolerance into standard databases would mean cryptographic verification of every internal packet — prohibitive overhead for no practical benefit in a controlled data center. For blockchains, you have no choice: proof-of-work and PBFT exist precisely because you cannot trust the nodes.

---

## 8. System Models

To reason mathematically about distributed algorithms, we need to formalize what kinds of failures we expect.

### Timing Models

| Model                     | Assumption                                                                            | Reality check                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Synchronous**           | Network delays, process pauses, and clock drift are all bounded by a known maximum    | Fairy tale for cloud computing; only realistic in hard real-time avionics                                    |
| **Asynchronous**          | No timing assumptions at all; no clocks, no timeouts                                  | Mathematically impossible to distinguish a crashed node from a slow one; almost no useful algorithms survive |
| **Partially synchronous** | System behaves synchronously _most_ of the time, but occasionally violates all bounds | This is reality: millisecond-level delays normally, multi-second GC pauses and network spikes occasionally   |

Practitioners design for **partial synchrony**.

### Safety vs. Liveness

Every distributed algorithm must satisfy two kinds of properties:

- **Safety** — nothing bad ever happens. Example: two nodes never hold the same lease simultaneously. If violated, the damage is permanent — you can't un-corrupt a database write. Safety must hold in all possible situations, even during network partitions. If the algorithm cannot guarantee a safe result, it must **halt and refuse** rather than produce a wrong answer.

- **Liveness** — something good eventually happens. Example: if a client sends a read request, the system eventually responds. Liveness is allowed to be conditional: "the system will respond _provided_ a majority of nodes are alive and the network eventually recovers." During a partition, liveness is temporarily suspended; safety is not.

The framework gives you a clean question for every feature: is this a safety requirement or a liveness requirement? The answer determines how strictly you must enforce it.

---

## 9. Deterministic Simulation Testing

You can prove your algorithm correct on a whiteboard, but eventually you write real code. And the bugs live in the implementation — in the exact microsecond timing of GC pauses, network drops, and thread scheduling across 10 machines.

Reproducing a concurrency bug in a live distributed environment is virtually impossible. **Deterministic Simulation Testing (DST)**, pioneered by FoundationDB, solves this by bringing the comforting single-node illusion back — for the purpose of testing.

### How DST Works

Replace every component that touches the real physical world:

- Real TCP stack → **mock simulated network**
- Real hardware clock → **simulated software clock**
- Thread schedulers, disk I/O → **simulated counterparts**
- Random number generators → **single seeded PRNG**

Run all 10 database instances inside a single-threaded process loop on a laptop. The simulator has absolute control: it dictates exactly when mock packets arrive, exactly when simulated processes pause, exactly what the clock reads. Because everything runs in software with no real I/O, you can fast-forward through millions of randomized failure scenarios — dropping every fifth packet, injecting a 10-second GC pause on node 3, jumping node 2's clock forward by an hour.

### The Magic of the Seed

If the simulator finds a bug at simulated hour 45 — say, a bizarre sequence of 100 packet drops triggers a split-brain data corruption — it stops and records the **random seed** that produced that exact sequence. You hand the developer the seed. They run the simulation on their laptop with that seed, watch the bug happen deterministically, attach a debugger, step through the code, and fix it.

DST turns the unpredictable, chaotic nightmare of distributed failures into a repeatable, debuggable, single-node puzzle.

---

## Summary

| Failure domain        | The problem                                                                     | The defense                                                            |
| --------------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Network               | Packets drop silently; client can't distinguish lost request from lost response | Timeouts (statistical guess); retry with idempotency                   |
| Flow control          | Fast sender overwhelms slow receiver                                            | TCP sliding window (backpressure); drop strategy for UDP               |
| Clock drift           | Quartz drifts with temperature; NTP assumes symmetric routing; leap seconds     | Monotonic clocks for durations; TrueTime intervals for global ordering |
| Process pauses        | GC, disk I/O, VM migration freeze application without its knowledge             | Fencing tokens; make the storage layer reject stale writes             |
| Byzantine nodes       | Actively malicious nodes fabricate authority                                    | PBFT / proof-of-work (only for untrusted environments)                 |
| Algorithm correctness | Partial synchrony makes proofs hard                                             | Safety + liveness decomposition; DST for implementation testing        |

The next time you see a spinning wheel in an app, ask yourself: is the network congested, did a database node pause for GC, or is the backend fending off a zombie holding an expired fencing token? The reality behind that wheel is a ballet of distributed chaos — hopefully protected by a solid quorum and rigorous simulation testing.
