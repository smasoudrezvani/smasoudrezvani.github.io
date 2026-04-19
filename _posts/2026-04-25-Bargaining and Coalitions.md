---
layout: post
title: "Part 6: Cooperation and Division – Bargaining Theory and Coalitional Games"
date: 2026-04-25
# description: "Nash Bargaining Solution, Rubinstein alternating offers, the core, and the Shapley Value in cooperative game theory."
tags: game-theory economics mathematics bargaining shapley-value cooperative-games
categories: ai-research
chart:
  echarts: true
---

*If you missed the previous chapter, start here: [Part 5: Bayesian Games and Incomplete Information](https://smasoudrezvani.github.io/blog/2026/Bayesian-Games/)*

Welcome to the final chapter of our game theory series! In Parts 2–5, we studied **non-cooperative games**: players make decisions independently, and agreements can't be enforced. Today we shift to **cooperative game theory**, where players can form binding agreements, communicate freely, and jointly decide how to split the gains from cooperation.

Two central questions arise: **How much will the whole group cooperate?** and **How should the gains be distributed?**

> ##### REFERENCE NOTE
> The concepts and mathematics in this post are drawn from Chapters 7 and 8 of *An Introduction to Game Theory* by Martin J. Osborne (Oxford University Press, 2003).
{: .block-tip }

---

## 1. Bargaining: Two Players, One Pie

The simplest cooperative situation: two players must agree on how to split a **surplus** of size 1. If they agree on $$(u_1, u_2)$$ with $$u_1 + u_2 \leq 1$$, they each get their respective share. If they **disagree**, they each receive a **disagreement payoff** $$(d_1, d_2)$$ (what they can guarantee themselves outside the negotiation).

The **feasible set** $$S \subset \mathbb{R}^2$$ is the set of all achievable utility pairs (often a convex, compact region). The **disagreement point** is $$d = (d_1, d_2) \in S$$.

A **bargaining solution** is a function $$f: (S, d) \mapsto (u_1^*, u_2^*)$$ selecting one point from $$S$$.

---

## 2. The Nash Bargaining Solution

Rather than modeling the negotiation process explicitly, John Nash (1950) proposed an **axiomatic approach**: list the properties any reasonable solution should satisfy, then show these uniquely determine a solution.

**The Four Nash Axioms:**

1. **Efficiency (PAR):** $$f(S,d)$$ lies on the Pareto frontier of $$S$$. There's no outcome in $$S$$ that makes both players strictly better off.
2. **Symmetry (SYM):** If the game is symmetric (swapping the players produces the same game), then both players receive the same payoff: $$u_1^* = u_2^*$$.
3. **Scale Invariance (SCI):** The solution is unaffected by positive affine transformations of utility. Rescaling payoffs doesn't change the outcome.
4. **Independence of Irrelevant Alternatives (IIA):** If $$f(S,d) \in S' \subseteq S$$, then $$f(S',d) = f(S,d)$$. Shrinking the feasible set doesn't change the solution as long as the solution stays feasible.

**Nash's Theorem:** These four axioms uniquely characterize the **Nash Bargaining Solution**:

$$f^{NB}(S, d) = \underset{(u_1, u_2) \in S,\; u_i \geq d_i}{\text{argmax}} \; (u_1 - d_1)(u_2 - d_2)$$

> ##### THE CORE CONCEPT
> The Nash Bargaining Solution maximizes the **product of utility gains** over the disagreement point. Players cooperate to find the point that jointly maximizes the multiplicative "pie" above their outside options.
{: .block-tip }

**Geometric Intuition:** The Nash product $$(u_1 - d_1)(u_2 - d_2) = k$$ is a hyperbola in the $$(u_1, u_2)$$ plane. The Nash solution is the point on the Pareto frontier of $$S$$ that is tangent to the highest such hyperbola.

---

## 3. The Rubinstein Alternating Offers Game

The Nash axiomatic approach tells us *what* the outcome should be, but not *why* — the process of negotiation is a black box. Ariel Rubinstein (1982) filled this gap with an explicit sequential bargaining model.

**Setup:** Two players alternately propose how to split a pie of size 1. Player 1 proposes first.
* If the opponent **accepts**, the game ends and that split is implemented.
* If the opponent **rejects**, they make a counter-offer in the next period.
* Each period of delay costs: Player $$i$$'s payoff in period $$t$$ is discounted by $$\delta_i^t \in (0,1)$$.

**The Unique SPE:** By backward induction on the infinite horizon (the game ends in finite time in equilibrium), the unique Subgame Perfect Equilibrium has Player 1 offering and Player 2 **immediately accepting**:

$$u_1^* = \frac{1 - \delta_2}{1 - \delta_1 \delta_2}, \quad u_2^* = \frac{\delta_2(1 - \delta_1)}{1 - \delta_1 \delta_2}$$

**Key Implications:**

* **Patience pays:** A more patient player (higher $$\delta_i$$) receives a larger share.
* **No delay:** Rational players agree immediately — delay is costly and gains nothing.
* **Equal patience → equal split:** If $$\delta_1 = \delta_2 = \delta$$, as $$\delta \to 1$$, $$u_1^* \to \frac{1}{2}$$ — the 50/50 split.

> ##### THE CORE CONCEPT
> The Rubinstein solution **converges to the Nash Bargaining Solution** with equal disagreement payoffs as discount factors approach 1. The axiomatic approach and the strategic approach reach the same destination from opposite directions.
{: .block-tip }

---

## 4. Coalitional Games: Beyond Two Players

With more than two players, coalitions can form and split off. A **coalitional game** (in characteristic function form) is defined by:

* A player set $$N = \{1, 2, \ldots, n\}$$.
* A **characteristic function** $$v : 2^N \rightarrow \mathbb{R}$$, assigning a real value $$v(S)$$ to every coalition $$S \subseteq N$$, with $$v(\emptyset) = 0$$.

$$v(S)$$ is the **total payoff** that coalition $$S$$ can guarantee its members by cooperating, regardless of what the other players do.

A game is **superadditive** if $$v(S \cup T) \geq v(S) + v(T)$$ for all disjoint $$S, T$$ — bigger coalitions are (weakly) more productive. Superadditivity makes formation of the grand coalition $$N$$ efficient.

---

## 5. The Core

Given that the grand coalition forms, how should the total value $$v(N)$$ be split? A **payoff allocation** is a vector $$x = (x_1, \ldots, x_n)$$ with $$\sum_{i \in N} x_i = v(N)$$ (efficiency).

An allocation is **blocked** by coalition $$S$$ if the members of $$S$$ can all do better by splitting off:
$$\sum_{i \in S} x_i < v(S)$$

**Definition (Core):** The **core** of a coalitional game is the set of efficient allocations that are not blocked by any coalition:

$$\text{Core}(v) = \left\{ x \in \mathbb{R}^n : \sum_{i \in N} x_i = v(N), \; \sum_{i \in S} x_i \geq v(S) \; \forall S \subseteq N \right\}$$

**Properties:**
* The core may be **empty** — no allocation is stable against all possible defections.
* The core may be a **large set** — many allocations are stable.
* Balancedness conditions (Bondareva–Shapley theorem) characterize when the core is non-empty.

---

## 6. The Shapley Value: A Unique Fair Allocation

The core describes *stable* allocations but doesn't uniquely prescribe how to split the pie. Lloyd Shapley (1953) took the axiomatic approach, asking: what is the **uniquely fair** way to distribute the grand coalition's value?

**The Four Shapley Axioms:**

1. **Efficiency:** $$\sum_{i \in N} \phi_i(v) = v(N)$$.
2. **Symmetry:** If players $$i$$ and $$j$$ are interchangeable (swapping them doesn't change $$v$$), then $$\phi_i = \phi_j$$.
3. **Null Player:** If player $$i$$ contributes nothing to any coalition — $$v(S \cup \{i\}) = v(S)$$ for all $$S$$ — then $$\phi_i = 0$$.
4. **Additivity:** $$\phi_i(v + w) = \phi_i(v) + \phi_i(w)$$ for any two games $$v$$ and $$w$$.

**Shapley's Theorem:** These axioms uniquely determine the **Shapley Value**:

$$\phi_i(v) = \sum_{S \subseteq N \setminus \{i\}} \frac{|S|!\,(|N| - |S| - 1)!}{|N|!} \left[ v(S \cup \{i\}) - v(S) \right]$$

**Interpretation:** Consider all $$|N|!$$ orderings in which players could arrive to form the grand coalition. For each ordering, player $$i$$'s **marginal contribution** is $$v(S \cup \{i\}) - v(S)$$, where $$S$$ is the set of players who arrived before $$i$$. The Shapley Value is player $$i$$'s **average marginal contribution** across all orderings.

The weight $$\frac{|S|!(|N|-|S|-1)!}{|N|!}$$ is exactly the probability that, in a uniformly random ordering of all $$n$$ players, the set of players arriving before $$i$$ is exactly $$S$$.

---

## 7. A Worked Example

Three players: $$N = \{1, 2, 3\}$$. The characteristic function:

$$v(\{1\}) = 0, \; v(\{2\}) = 0, \; v(\{3\}) = 0$$
$$v(\{1,2\}) = 6, \; v(\{1,3\}) = 4, \; v(\{2,3\}) = 2, \; v(\{1,2,3\}) = 9$$

**Computing $$\phi_1$$:** We need the marginal contribution of Player 1 in each of the $$3! = 6$$ orderings:

| Ordering | Coalition before 1 | Marginal contribution |
| :--- | :---: | :---: |
| (1, 2, 3) | $$\emptyset$$ | $$v(\{1\}) - v(\emptyset) = 0$$ |
| (1, 3, 2) | $$\emptyset$$ | $$0$$ |
| (2, 1, 3) | $$\{2\}$$ | $$v(\{1,2\}) - v(\{2\}) = 6$$ |
| (3, 1, 2) | $$\{3\}$$ | $$v(\{1,3\}) - v(\{3\}) = 4$$ |
| (2, 3, 1) | $$\{2,3\}$$ | $$v(\{1,2,3\}) - v(\{2,3\}) = 7$$ |
| (3, 2, 1) | $$\{2,3\}$$ | $$7$$ |
{: .table .table-bordered .table-striped }

$$\phi_1 = \frac{1}{6}(0 + 0 + 6 + 4 + 7 + 7) = \frac{24}{6} = 4$$

Similarly: $$\phi_2 = \frac{1}{6}(6+6+0+0+2+2) \cdot \frac{1}{6}$$ ... working through all orderings yields $$\phi_2 = \frac{17}{6} \approx 2.83$$ and $$\phi_3 = \frac{13}{6} \approx 2.17$$. Check: $$4 + \frac{17}{6} + \frac{13}{6} = 4 + 5 = 9 = v(N)$$ ✓

---

## 8. Visualizing Shapley Values

```echarts
{
  "title": { "text": "Shapley Value Distribution (3-Player Example)", "left": "center" },
  "tooltip": { "trigger": "item", "formatter": "{b}: {c} ({d}%)" },
  "legend": { "orient": "vertical", "left": "left", "data": ["Player 1 (φ=4.00)", "Player 2 (φ=2.83)", "Player 3 (φ=2.17)"] },
  "series": [{
    "name": "Shapley Value",
    "type": "pie",
    "radius": "60%",
    "data": [
      { "value": 4.00, "name": "Player 1 (φ=4.00)", "itemStyle": {"color": "#2196F3"} },
      { "value": 2.83, "name": "Player 2 (φ=2.83)", "itemStyle": {"color": "#4CAF50"} },
      { "value": 2.17, "name": "Player 3 (φ=2.17)", "itemStyle": {"color": "#FF9800"} }
    ],
    "emphasis": { "itemStyle": { "shadowBlur": 10, "shadowColor": "rgba(0,0,0,0.5)" } }
  }]
}
```

---

## Series Summary

We have now traveled through the full landscape of game theory — from two players choosing simultaneously without a word spoken, all the way to large coalitions distributing joint gains by fairness axioms.

| Post | Topic | Core Concept |
| :--- | :--- | :--- |
| Part 1 | Introduction | The strategic interaction framework |
| Part 2 | Nash Equilibrium | Self-enforcing unilateral stability |
| Part 3 | Mixed Strategies | Randomization and the indifference principle |
| Part 4 | Extensive Games | Backward induction and credible threats (SPE) |
| Part 5 | Bayesian Games | Private types, BNE, and auction theory |
| Part 6 | Bargaining & Coalitions | Nash solution, Rubinstein, core, and Shapley value |
{: .table .table-bordered .table-striped }

Game theory is not just an abstract mathematical exercise. Its ideas appear in AI (multi-agent reinforcement learning, mechanism design for LLMs, RLHF), economics (auction design, contract theory), political science (voting theory, international relations), and evolutionary biology (evolutionarily stable strategies). The language of rational strategic interaction is universal.
