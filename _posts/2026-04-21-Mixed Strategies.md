---
layout: post
title: "Part 3: Embracing Randomness – Mixed Strategy Nash Equilibrium"
date: 2026-04-21
# description: "Why rational players sometimes randomize, how to compute mixed strategy equilibria, and the indifference principle."
tags: game-theory economics mathematics mixed-strategies nash-equilibrium
categories: game-theory
series: game-theory
chart:
  echarts: true
---

*If you missed the previous chapter, start here: [Part 2: Nash Equilibrium in Normal Form Games](https://smasoudrezvani.github.io/blog/2026/Nash-Equilibrium/)*

Welcome back! In Part 2, we saw that some games — like the Battle of the Sexes — have multiple Nash Equilibria in pure strategies. Others, like Matching Pennies, have *none* at all. Today we resolve this puzzle by expanding the strategy space to allow **randomization**.

> ##### REFERENCE NOTE
> The concepts and mathematics in this post are drawn from Chapter 4 of *An Introduction to Game Theory* by Martin J. Osborne (Oxford University Press, 2003).
{: .block-tip }

---

## 1. Why Pure Strategies Are Not Enough

Consider **Matching Pennies**: two players each put a coin on a table. If both show Heads or both show Tails (they **match**), Player 1 wins (+1, -1). If they differ, Player 2 wins (-1, +1).

|  | **H** | **T** |
| :--- | :---: | :---: |
| **H** | (1, −1) | (−1, 1) |
| **T** | (−1, 1) | (1, −1) |
{: .table .table-bordered .table-striped }

Player 1 wants to match. Player 2 wants to mismatch. Check all four pure profiles:
* **(H, H):** Player 2 wants to switch to T. Not a NE.
* **(H, T):** Player 1 wants to switch to T. Not a NE.
* **(T, H):** Player 1 wants to switch to H. Not a NE.
* **(T, T):** Player 2 wants to switch to H. Not a NE.

No pure strategy Nash Equilibrium exists. The game is **zero-sum** — whatever one player gains, the other loses — and it is inherently competitive and unpredictable. The solution is randomization.

---

## 2. Mixed Strategies: Formalizing Randomness

**Definition:** A **mixed strategy** $$\sigma_i$$ for player $$i$$ is a probability distribution over their pure strategies $$A_i$$:

$$\sigma_i : A_i \rightarrow [0,1], \quad \sum_{a_i \in A_i} \sigma_i(a_i) = 1$$

We write $$\sigma_i(a_i)$$ for the probability assigned to action $$a_i$$. A pure strategy is a special case where $$\sigma_i(a_i) = 1$$ for one action and 0 for all others.

The set of all mixed strategies for player $$i$$ is the **simplex** $$\Delta(A_i)$$. For a two-action game $$A_i = \{H, T\}$$, this is just the interval $$[0,1]$$ parameterized by $$p = \sigma_i(H)$$.

The **support** of a mixed strategy is the set of pure actions played with positive probability:
$$\text{supp}(\sigma_i) = \{a_i \in A_i : \sigma_i(a_i) > 0\}$$

---

## 3. Expected Payoffs

When players mix, the outcome is random. We evaluate strategies using **expected payoffs**.

**Definition:** Given a mixed strategy profile $$\sigma = (\sigma_1, \ldots, \sigma_n)$$, player $$i$$'s expected payoff is:

$$U_i(\sigma) = \sum_{a \in A} \left( \prod_{j \in N} \sigma_j(a_j) \right) u_i(a)$$

This is just the probability-weighted sum of $$u_i$$ over all pure action profiles.

For the two-player case, if Player 1 plays $$H$$ with probability $$p$$ and Player 2 plays $$H$$ with probability $$q$$:

$$U_1(p, q) = pq \cdot (1) + p(1-q) \cdot (-1) + (1-p)q \cdot (-1) + (1-p)(1-q) \cdot (1)$$

$$U_1(p, q) = 2pq - 2p - 2q + 2(1-p)(1-q) + \ldots = 4pq - 2p - 2q + 1$$

Simplifying: $$U_1(p, q) = (2q - 1)(2p - 1)$$

---

## 4. The Indifference Principle

How do we find a mixed strategy Nash Equilibrium? The key insight is the **indifference condition**.

> ##### THE CORE CONCEPT
> In any Nash Equilibrium, if player $$i$$ mixes over a set of pure strategies $$S \subseteq A_i$$, then every strategy in $$S$$ must yield the **same expected payoff**. If one action were strictly better, the player would shift all probability onto it — contradicting mixing.
{: .block-tip }

This gives us a powerful algorithm: set up equations that make the *opponent* indifferent among their mixed strategies, then solve for the mixing probabilities.

### Solving Matching Pennies

Let Player 1 mix with $$P(\text{H}) = p$$, Player 2 mix with $$P(\text{H}) = q$$.

**Making Player 1 indifferent** (finding $$q^*$$):

$$U_1(H, q) = q \cdot 1 + (1-q)(-1) = 2q - 1$$
$$U_1(T, q) = q \cdot (-1) + (1-q)(1) = 1 - 2q$$

Setting equal: $$2q-1 = 1-2q \Rightarrow 4q = 2 \Rightarrow q^* = \frac{1}{2}$$

**Making Player 2 indifferent** (finding $$p^*$$):

By symmetry (it is a zero-sum game), $$p^* = \frac{1}{2}$$.

The unique Nash Equilibrium is $$(\sigma_1^*, \sigma_2^*) = \left(\frac{1}{2}H + \frac{1}{2}T, \; \frac{1}{2}H + \frac{1}{2}T\right)$$.

---

## 5. Mixed Equilibrium in the Battle of the Sexes

The Battle of the Sexes also has a mixed strategy Nash Equilibrium alongside its two pure ones. Player 1 prefers Opera (O), Player 2 prefers Football (F).

|  | **O** | **F** |
| :--- | :---: | :---: |
| **O** | (2, 1) | (0, 0) |
| **F** | (0, 0) | (1, 2) |
{: .table .table-bordered .table-striped }

Let $$p = \sigma_1(O)$$ and $$q = \sigma_2(O)$$.

**Making Player 2 indifferent over O vs. F** (finding $$p^*$$):

$$U_2(p, O) = p \cdot 1 + (1-p) \cdot 0 = p$$
$$U_2(p, F) = p \cdot 0 + (1-p) \cdot 2 = 2(1-p)$$

Setting equal: $$p = 2(1-p) \Rightarrow 3p = 2 \Rightarrow p^* = \frac{2}{3}$$

**Making Player 1 indifferent** (finding $$q^*$$):

$$U_1(O, q) = 2q, \quad U_1(F, q) = 1-q$$

Setting equal: $$2q = 1-q \Rightarrow q^* = \frac{1}{3}$$

The mixed NE is $$\left(\frac{2}{3}O + \frac{1}{3}F, \; \frac{1}{3}O + \frac{2}{3}F\right)$$. Crucially, the expected payoffs at this equilibrium are lower for *both* players than at either pure strategy equilibrium — the "coordination failure" has a real cost.

---

## 6. Rock-Paper-Scissors

In Rock-Paper-Scissors (R, P, S), no pure strategy equilibrium exists. The unique NE is full mixing:

$$\sigma^* = \left(\frac{1}{3}R, \frac{1}{3}P, \frac{1}{3}S\right) \text{ for each player}$$

The proof is by the indifference principle: if the opponent mixes uniformly, each pure action yields the same expected payoff (0), so the player is indifferent — and any mixture, including the uniform one, is a best response.

---

## 7. Visualizing Mixed Strategy Payoffs

The chart below shows Player 1's expected payoff in Matching Pennies as a function of their mixing probability $$p$$, for three fixed strategies of Player 2.

```echarts
{
  "title": { "text": "Matching Pennies: Player 1 Expected Payoff vs. p (mixing prob.)", "left": "center" },
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["q=0 (always T)", "q=0.5 (mixed)", "q=1 (always H)"], "bottom": 0 },
  "xAxis": { "type": "category", "name": "p = P(Heads)", "data": ["0", "0.1", "0.2", "0.3", "0.4", "0.5", "0.6", "0.7", "0.8", "0.9", "1"] },
  "yAxis": { "type": "value", "name": "Expected Payoff U_1" },
  "series": [
    { "name": "q=0 (always T)", "type": "line", "data": [1, 0.8, 0.6, 0.4, 0.2, 0, -0.2, -0.4, -0.6, -0.8, -1], "lineStyle": {"width": 3, "color": "#f44336"} },
    { "name": "q=0.5 (mixed)", "type": "line", "data": [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0], "lineStyle": {"width": 3, "color": "#4CAF50", "type": "dashed"} },
    { "name": "q=1 (always H)", "type": "line", "data": [-1, -0.8, -0.6, -0.4, -0.2, 0, 0.2, 0.4, 0.6, 0.8, 1], "lineStyle": {"width": 3, "color": "#2196F3"} }
  ]
}
```

When Player 2 mixes with $$q=0.5$$, Player 1's payoff is **flat at 0** regardless of $$p$$ — confirming indifference. Any $$p$$ is a best response when $$q = 0.5$$.

---

## Summary

| Concept | Key Idea |
| :--- | :--- |
| **Mixed Strategy** | Probability distribution over pure actions |
| **Expected Payoff** | Probability-weighted average of pure-strategy payoffs |
| **Mixed NE** | Each player best-responds to others' mixed strategies |
| **Indifference Condition** | Mixing player is indifferent among all actions in their support |
| **Nash's Theorem** | Every finite game has at least one NE (possibly mixed) |
{: .table .table-bordered .table-striped }

---

## What's Next?

So far, players have moved simultaneously — nobody sees anyone else's choice before acting. But many real-world interactions are **sequential**: one player moves first, the other observes and responds.

In Part 4, we introduce **Extensive Form Games**, game trees, and the powerful concept of **Subgame Perfect Equilibrium** — which lets us identify which Nash Equilibria are backed by credible strategic commitments.
