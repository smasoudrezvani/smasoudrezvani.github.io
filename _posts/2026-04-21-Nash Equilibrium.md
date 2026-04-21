---
layout: post
title: "Part 2: The Cornerstone – Nash Equilibrium in Normal Form Games"
date: 2026-04-21
# description: "A rigorous deep dive into strategic form games, dominant strategies, best responses, and the Nash Equilibrium."
tags: game-theory economics mathematics nash-equilibrium prisoner-dilemma
categories: ai-research
chart:
  echarts: true
---

*If you missed the beginning of our series, start here: [Part 1: A Comprehensive Introduction to Game Theory](https://smasoudrezvani.github.io/blog/2026/Game-theory-intro/)*

Welcome back! In Part 1, we mapped out the entire landscape of game theory. Now we roll up our sleeves and start at the very foundation: the **strategic (normal) form game** and its central solution concept, the **Nash Equilibrium**.

> ##### REFERENCE NOTE
> The concepts and mathematics in this post are drawn from Chapters 1–3 of *An Introduction to Game Theory* by Martin J. Osborne (Oxford University Press, 2003).
{: .block-tip }

---

## 1. What Is a Strategic Game?

Before we can solve a game, we need to describe it precisely. A **strategic game** (or normal form game) is a model for situations where:

1. A finite set of **players** each choose a strategy *simultaneously* and *independently*.
2. Every combination of strategies produces a well-defined **payoff** for each player.

**Formal Definition:**

A strategic game is a triple $$\langle N, \{A_i\}_{i \in N}, \{u_i\}_{i \in N} \rangle$$ consisting of:

* A finite set of players $$N = \{1, 2, \ldots, n\}$$.
* For each player $$i \in N$$, a nonempty set of **actions** $$A_i$$ (also called pure strategies).
* For each player $$i \in N$$, a **payoff function** $$u_i : A \rightarrow \mathbb{R}$$, where $$A = A_1 \times A_2 \times \cdots \times A_n$$ is the set of all action profiles.

An **action profile** is a tuple $$a = (a_1, a_2, \ldots, a_n)$$ specifying one action for each player. We write $$a_{-i}$$ for the actions of all players *except* player $$i$$.

---

## 2. Representing a Game: The Payoff Matrix

For two-player games with finite action sets, the most compact representation is the **payoff matrix** (also called a bimatrix). Each row corresponds to an action of Player 1, each column to an action of Player 2, and each cell contains the pair $$(u_1, u_2)$$.

### The Prisoner's Dilemma

Two suspects are interrogated separately. Each can either **Cooperate (C)** (stay silent) or **Defect (D)** (betray the other). The payoffs are (years in prison, negated so higher is better):

|  | **C** | **D** |
| :--- | :---: | :---: |
| **C** | (3, 3) | (0, 5) |
| **D** | (5, 0) | (1, 1) |
{: .table .table-bordered .table-striped }

Here, $$(3,3)$$ means both get 3 years of freedom, $$(0, 5)$$ means Player 1 gets 0 and Player 2 gets 5. The tragedy: both players choose D, landing at $$(1,1)$$, even though both prefer $$(3,3)$$.

---

## 3. Dominant Strategies

Sometimes one action is unambiguously best for a player — no matter what the opponent does.

**Definition:** Action $$a_i$$ **strictly dominates** action $$a_i'$$ for player $$i$$ if:
$$u_i(a_i, a_{-i}) > u_i(a_i', a_{-i}) \quad \forall a_{-i} \in A_{-i}$$

A rational player will **never** play a strictly dominated action. In the Prisoner's Dilemma, D strictly dominates C for both players: regardless of what the opponent does, defecting always yields a strictly higher payoff.

> ##### THE CORE CONCEPT
> When every player has a **dominant strategy**, the outcome is uniquely determined. The result is called a **dominant strategy equilibrium** — and it is a Nash Equilibrium.
{: .block-tip }

**Iterated Elimination of Strictly Dominated Strategies (IESDS):** Even when dominant strategies don't exist for everyone, we can iteratively remove dominated actions. If the process leads to a unique outcome, that outcome is the unique Nash Equilibrium.

---

## 4. Best Response and Nash Equilibrium

For most games, no player has a dominant strategy. We need a more general solution concept.

**Definition (Best Response):** Player $$i$$'s **best response** to the strategy profile $$a_{-i}$$ of the others is:
$$B_i(a_{-i}) = \underset{a_i \in A_i}{\text{argmax}} \; u_i(a_i, a_{-i})$$

The best response is the optimal action *given what everyone else does*. A Nash Equilibrium is a profile where every player is simultaneously playing their best response.

**Definition (Nash Equilibrium):** A strategy profile $$a^* = (a_1^*, \ldots, a_n^*)$$ is a **Nash Equilibrium** if, for every player $$i \in N$$:

$$u_i(a_i^*, a_{-i}^*) \geq u_i(a_i, a_{-i}^*) \quad \forall a_i \in A_i$$

Equivalently: $$a_i^* \in B_i(a_{-i}^*)$$ for all $$i$$.

> ##### THE CORE CONCEPT
> In a Nash Equilibrium, no player has a **unilateral incentive to deviate**. It is a *self-enforcing* agreement: if all players expect equilibrium play, then playing the equilibrium action is individually rational.
{: .block-tip }

---

## 5. Canonical Examples

### The Battle of the Sexes

A couple wants to spend an evening together. Player 1 prefers **Opera (O)**, Player 2 prefers **Football (F)**, but both prefer being together over being apart.

|  | **O** | **F** |
| :--- | :---: | :---: |
| **O** | (2, 1) | (0, 0) |
| **F** | (0, 0) | (1, 2) |
{: .table .table-bordered .table-striped }

Checking best responses:
* If Player 2 plays O: Player 1 prefers O (payoff 2 > 0). ✓
* If Player 2 plays F: Player 1 prefers F (payoff 1 > 0). ✓

By symmetry, both **(O, O)** and **(F, F)** are Nash Equilibria in pure strategies. There is also a third, mixed strategy equilibrium (covered in Part 3).

### The Stag Hunt

Two hunters can cooperate to hunt a **Stag (S)** (large reward if *both* cooperate) or individually hunt a **Hare (H)** (small but certain reward).

|  | **S** | **H** |
| :--- | :---: | :---: |
| **S** | (4, 4) | (0, 3) |
| **H** | (3, 0) | (3, 3) |
{: .table .table-bordered .table-striped }

Both **(S, S)** and **(H, H)** are Nash Equilibria. The **(S, S)** equilibrium is **Pareto superior** (both players prefer it), but **(H, H)** is **risk dominant** — it is safe regardless of what the other player does. The Stag Hunt models a deep tension between **social optimality** and **individual risk aversion**.

---

## 6. Nash's Existence Theorem

Do Nash Equilibria always exist? Not always in pure strategies — we will see a counterexample in Part 3. But Nash proved something powerful:

> **Nash's Theorem (1950):** Every finite strategic game (finite players, finite action sets) has at least one Nash Equilibrium in **mixed strategies**.

The proof uses **Kakutani's fixed-point theorem**: define the combined best-response correspondence $$B(a) = B_1(a_{-1}) \times \cdots \times B_n(a_{-n})$$. Under mixed strategies, this correspondence is convex-valued and upper-hemicontinuous, so by Kakutani's theorem, it has a fixed point — which is exactly a Nash Equilibrium.

---

## Visualizing Payoffs

Below is a chart of each player's payoff under both pure strategy profiles in the Prisoner's Dilemma, illustrating the tension at the heart of the game.

```echarts
{
  "title": { "text": "Prisoner's Dilemma: Payoffs by Outcome", "left": "center" },
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["Player 1 Payoff", "Player 2 Payoff"], "bottom": 0 },
  "xAxis": { "type": "category", "data": ["(C,C)", "(C,D)", "(D,C)", "(D,D)"] },
  "yAxis": { "type": "value", "name": "Payoff" },
  "series": [
    { "name": "Player 1 Payoff", "type": "bar", "data": [3, 0, 5, 1], "itemStyle": {"color": "#2196F3"} },
    { "name": "Player 2 Payoff", "type": "bar", "data": [3, 5, 0, 1], "itemStyle": {"color": "#4CAF50"} }
  ]
}
```

---

## What's Next?

We have established the backbone of game theory: the strategic form game and its Nash Equilibrium. But we saw that some games — like Battle of the Sexes — have multiple equilibria, and others have none in pure strategies.

In Part 3, we will introduce **Mixed Strategy Nash Equilibrium**, learn how to compute it, and see why randomization is a fundamental, rational behavior — not a sign of indecision!
