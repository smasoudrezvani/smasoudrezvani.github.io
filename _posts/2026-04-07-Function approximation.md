---
layout: post
title: "Part 5: Breaking the Table – Function Approximation"
date: 2026-04-07
# description: "Overcoming the Curse of Dimensionality in RL using neural networks, SGD, and avoiding the Deadly Triad."
tags: reinforcement-learning machine-learning ai deep-rl
categories: reinforcement-learning
series: reinforcement-learning
chart:
  echarts: true
---

_If you missed our deep dive into on-policy vs. off-policy algorithms, start here: [Walking the Cliff's Edge: SARSA vs. Q-Learning](https://smasoudrezvani.github.io/blog/2026/Sarsa-vs.-qlearning/)_

Welcome back! In Part 3, we solved Markov Decision Processes using Tabular Methods (Dynamic Programming, Monte Carlo, and TD Learning).

But tabular methods harbor a fatal flaw: **The Curse of Dimensionality**.

If you are teaching an agent to play Tic-Tac-Toe, there are about 5,478 valid states. A computer can easily store a table with 5,478 rows. But what if the agent is playing Backgammon ($$10^{20}$$ states), Go ($$10^{170}$$ states), or driving a car using raw camera pixels?

You cannot build a table larger than the number of atoms in the universe. More importantly, even if you could, the agent would never have enough time to visit every single state to learn its value. To solve real-world problems, we must abandon the table. We need our agent to experience a fraction of the possible states and _generalize_ what it learns to states it has never seen before.

> ##### REFERENCE NOTE
>
> The transition from explicit tables to parameterized equations marks the shift from Part I to Part II in Sutton and Barto's _Reinforcement Learning: An Introduction_.
> {: .block-tip }

---

### 1. The Solution: Function Approximation

Instead of storing the value of every state in an explicit lookup table, we use a mathematical function to estimate the value of a state based on its features.

We define the value function as $$\hat{v}(s, \mathbf{w})$$, where $$\mathbf{w}$$ is a vector of weights (parameters).

- In the 1990s, this function was often a linear equation.
- Today, this function is almost always a Deep Neural Network (hence, **Deep RL**).

When the agent visits a state and receives a reward, it no longer updates a specific row in a table. Instead, it tweaks the weights $$\mathbf{w}$$ so that the function's output gets slightly closer to the observed reward.

#### The Superpower: Generalization

If a tabular agent learns that a car swerving into its lane is bad, it only knows that _exact_ pixel arrangement is bad. If the swerving car is one inch to the left, it's a completely new state, and the agent has to learn the danger all over again.

A function approximator extracts _features_ (like the general shape and trajectory of a car). When it updates its weights to penalize the swerving car, the estimated value of all mathematically similar states drops simultaneously. It learns from one state and instantly applies that knowledge to millions of others.

---

### 2. The Math Engine: Stochastic Gradient Descent (SGD)

How do we actually update these weights? We borrow the workhorse of machine learning: **Stochastic Gradient Descent (SGD)**.

Our goal is to minimize the error between our current guess $$\hat{v}(S_t, \mathbf{w}_t)$$and the actual target$$U_t$$ (which could be the true return from Monte Carlo, or a bootstrapped TD estimate).

**The General SGD Update Rule:**
$$\mathbf{w}_{t+1} \doteq \mathbf{w}_t + \alpha \left[ U_t - \hat{v}(S_t, \mathbf{w}_t) \right] \nabla \hat{v}(S_t, \mathbf{w}_t)$$

**Breaking it down:**

- **$$\mathbf{w}_t$$**: Our current weights.
- **$$\alpha$$**: The learning rate (how big of a step we take).
- **$$\left[ U_t - \hat{v}(S_t, \mathbf{w}_t) \right]$$**: The error. The difference between what we thought would happen and what actually happened.
- **$$\nabla \hat{v}(S_t, \mathbf{w}_t)$$**: The gradient. This is a vector of partial derivatives telling us exactly which direction to nudge each individual weight to reduce the error.

> ##### DEEP Q-NETWORKS (DQN)
>
> When you apply this concept to Q-Learning and use a deep neural network as the function approximator, you get a Deep Q-Network (DQN). This is the exact architecture DeepMind used in 2015 to achieve superhuman performance on Atari games, kicking off the modern Deep RL revolution.
> {: .block-tip }

---

### 3. The Catch: The Deadly Triad

Moving from tables to neural networks isn't free. Sutton and Barto dedicate significant time to a phenomenon called the **Deadly Triad**.

RL training becomes famously unstable, and can outright diverge (the weights explode to infinity), if you combine these three things simultaneously:

1. **Function Approximation:** Using a neural network or linear model instead of a precise table.
2. **Bootstrapping:** Updating guesses based on other guesses (like TD Learning and Q-Learning do).
3. **Off-Policy Training:** Learning about the optimal policy while following an exploratory behavior policy (like Q-Learning does).

Modern algorithms (like DQN's use of "Target Networks" and "Experience Replay") are largely just clever engineering tricks designed to stabilize this Deadly Triad.

---

### See Generalization in Action

The difference between Tabular learning and Function Approximation is highly visual. Imagine the agent experiences a highly rewarding event at State 50.

Notice how the **Tabular** update only changes the exact state visited (a single isolated spike). In contrast, the **Function Approximation** update smoothly generalizes the reward to mathematically similar, unseen states nearby!

```echarts
{
  "title": { "text": "State Value Update: Tabular vs Function Approximation", "left": "center" },
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["Tabular Update", "Function Approximation Update"], "bottom": 0 },
  "xAxis": { "type": "category", "name": "State Space", "data": ["10", "20", "30", "40", "50", "60", "70", "80", "90"] },
  "yAxis": { "type": "value", "name": "Estimated Value" },
  "series": [
    {
      "name": "Tabular Update",
      "type": "bar",
      "data": [0, 0, 0, 0, 1.0, 0, 0, 0, 0],
      "itemStyle": {"color": "#f44336"}
    },
    {
      "name": "Function Approximation Update",
      "type": "line",
      "smooth": true,
      "data": [0.01, 0.05, 0.2, 0.6, 1.0, 0.6, 0.2, 0.05, 0.01],
      "lineStyle": {"width": 3, "color": "#2196F3"},
      "areaStyle": {"opacity": 0.2, "color": "#2196F3"}
    }
  ]
}
```
