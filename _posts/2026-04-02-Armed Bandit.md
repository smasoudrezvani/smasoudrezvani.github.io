---
layout: post
title: "Part 2: The Multi-Armed Bandit – Mastering the Art of Choices"
date: 2026-04-02
# description: "Exploring the fundamental dilemma of Reinforcement Learning: Exploration vs. Exploitation using the k-Armed Bandit problem."
tags: reinforcement-learning machine-learning ai multi-armed-bandit
categories: ai-research
chart:
  echarts: true
---

*If you missed the beginning of our series, start here: [Part 1: Mapping the Maze](https://smasoudrezvani.github.io/blog/2026/RL-intro/)*

Welcome back to our journey through the maze of Reinforcement Learning (RL)! In Part 1, we mapped out the big picture. Now, we are rolling up our sleeves and starting at the very beginning.

Before an AI can learn to play chess or navigate a complex environment, it must learn how to make a single choice among multiple options. To understand this, we turn to a classic thought experiment in probability and RL: The $k$-Armed Bandit Problem.

> ##### REFERENCE NOTE
> The concepts and mathematics in this post are drawn directly from Chapter 2 of *Reinforcement Learning: An Introduction* by Sutton and Barto.
{: .block-tip }

### The Setup: A Casino Dilemma

Imagine you are in a casino standing in front of $k$ slot machines (historically known as "one-armed bandits"). Each machine pays out a different average reward, but you have no idea what those averages are. Your goal is to maximize your total winnings over a limited number of pulls.

Every time you pull an arm, you face the most fundamental dilemma in all of Reinforcement Learning: **Exploration vs. Exploitation**.

* **Exploitation:** Do you pull the arm that has given you the highest payout so far?
* **Exploration:** Do you try an arm you haven't pulled much, hoping it has an even higher payout?

If you only exploit, you might get stuck pulling a mediocre machine forever, never realizing the machine next to it is a jackpot. If you only explore, you waste your time pulling bad machines and never cash in on the good ones.

Let's look at how RL agents solve this mathematically.

### The Foundation: Action-Value Methods

To make good decisions, our agent needs to estimate the value of each action.
* We denote the true (but hidden) value of an action $a$ as $q_*(a)$.
* We denote the agent's estimated value of action $a$ at time step $t$ as $Q_t(a)$.

The easiest way to estimate $Q_t(a)$ is to average the rewards actually received when taking action $a$:

$$Q_t(a) \doteq \frac{\text{sum of rewards when } a \text{ taken prior to } t}{\text{number of times } a \text{ taken prior to } t}$$

Now that the agent has estimates, how does it choose what to do?

### Method 1: The Greedy Approach

The simplest strategy is to always pick the action with the highest estimated value. This is called a greedy action.

**The Math:** $$A_t \doteq \underset{a}{\text{argmax}} \, Q_t(a)$$

**The Flaw:** Pure exploitation. If the agent gets lucky on a bad machine early on, it might stick with it forever and completely ignore a better machine that happened to give a poor payout on the first pull.

### Method 2: $\epsilon$-Greedy (Epsilon-Greedy)

To fix the flaw of the pure greedy method, we force the agent to explore occasionally. We introduce a tiny probability, $\epsilon$ (epsilon), say 0.1 (10% of the time).

**The Strategy:** Most of the time ($1-\epsilon$), the agent exploits and picks the highest $Q_t(a)$. But occasionally ($\epsilon$), it picks an action completely at random, regardless of its value estimates.

**The Advantage:** Over time, $\epsilon$-greedy guarantees that every action will be sampled enough times to accurately estimate its true value, preventing the agent from getting permanently stuck on a bad choice.

### Method 3: Upper Confidence Bound (UCB)

$\epsilon$-greedy explores blindly—when it explores, it picks randomly. Can we be smarter? Yes. 

Upper Confidence Bound (UCB) explores based on *uncertainty*. It favors actions that might be great, either because their estimated value is high, or because we haven't tried them enough to know for sure.

**The Math:**
$$A_t \doteq \underset{a}{\text{argmax}} \left[ Q_t(a) + c \sqrt{\frac{\ln t}{N_t(a)}} \right]$$

**Breaking it down:**
* $Q_t(a)$: The current estimated value (Exploitation).
* $c$: A parameter you set to control how much you want to explore.
* $\sqrt{\frac{\ln t}{N_t(a)}}$: The uncertainty bonus (Exploration). $t$ is the total number of steps taken, and $N_t(a)$ is how many times action $a$ has been picked. If an action hasn't been picked much, $N_t(a)$ is small, making this fraction massive. This forces the agent to try it!

---

### Visualizing the Results

Below is a chart visualizing the average performance of these three algorithms over 1,000 steps on a 10-Armed Bandit problem. Notice how a purely greedy algorithm often gets stuck at a sub-optimal local maximum, while UCB efficiently explores the uncertainty to find the optimal arm and achieve the highest average reward!

```echarts
{
  "title": { "text": "Algorithm Performance: Average Reward over 1000 Steps", "left": "center" },
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["Greedy", "ε-Greedy (ε=0.1)", "UCB (c=2)"], "bottom": 0 },
  "xAxis": { "type": "category", "name": "Steps", "data": ["10", "50", "100", "200", "500", "1000"] },
  "yAxis": { "type": "value", "name": "Average Reward" },
  "series": [
    { "name": "Greedy", "type": "line", "data": [0.5, 0.9, 1.0, 1.05, 1.05, 1.05], "lineStyle": {"width": 3, "color": "#f44336"} },
    { "name": "ε-Greedy (ε=0.1)", "type": "line", "data": [0.2, 0.6, 0.9, 1.1, 1.3, 1.35], "lineStyle": {"width": 3, "color": "#2196F3"} },
    { "name": "UCB (c=2)", "type": "line", "data": [0.1, 0.5, 0.8, 1.25, 1.45, 1.52], "lineStyle": {"width": 3, "color": "#4CAF50"} }
  ]
}
```

<!-- *(Note: To embed a fully playable JS simulation into `al-folio` in the future, you can code the logic in a separate HTML file and use the `<iframe src="{{ '/assets/plotly/demo.html' | relative_url }}"></iframe>` tag, just like the Distill layout example!)* -->

---

### What’s Next?

Bandit algorithms are fantastic for making single decisions in a stationary setup. But what happens when taking an action changes the world around you? What if pulling a lever doesn't just give you a reward, but transports you to a completely different room with a new set of levers?

In Part 3, we will introduce Markov Decision Processes (MDPs), where we step out of the casino and into environments that evolve based on our choices.