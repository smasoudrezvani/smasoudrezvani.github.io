---
layout: post
title: "Part 7: The Complete Alignment Pipeline — From SFT to Advanced RL"
date: 2026-04-15
description: "A complete guide to how PPO, DPO, and GRPO transform language models from pattern copiers into reasoning agents."
tags: reinforcement-learning machine-learning ai ppo dpo grpo
categories: ai-research
---

*If you missed our previous discussion on Policy Gradients and Actor-Critic architectures, start here: [Part 6: Cutting Out the Middleman – Policy Gradient Methods](https://smasoudrezvani.github.io/blog/2026/Policy-Gradient/)*

To truly master Proximal Policy Optimization (PPO), Direct Preference Optimization (DPO), and Group Relative Policy Optimization (GRPO), we need to understand exactly where they fit in the lifecycle of a Large Language Model (LLM). 

Before any reinforcement learning happens, a model goes through **Pretraining** (learning general language from massive unlabeled data) and **Supervised Fine-Tuning (SFT)** (imitating curated human demonstrations to learn how to follow instructions). This SFT model becomes our frozen "Reference Model" ($$\pi_{\text{ref}}$$).

However, SFT is insufficient on its own. It suffers from static behavior (it struggles to generalize outside its training data) and it has no mechanism for ranking outputs when multiple valid options exist—it just imitates. Reinforcement Learning Post-Training transforms the model from a pattern copier into an agent that optimizes for what humans actually want.

### Standard Terminology

Before diving into the algorithms, here is the vocabulary that unifies them all in the context of LLMs:

| Term | Meaning in LLM Context |
| :--- | :--- |
| **State ($$s_t$$)** | The prompt + all tokens generated so far. |
| **Action ($$a_t$$)** | The next token being generated. |
| **Policy ($$\pi_\theta$$)** | The LLM itself — a probability distribution over tokens. |
| **Trajectory ($$\tau$$)** | The full prompt-to-EOS sequence. |
| **Reward ($$R$$)** | A scalar score evaluating the full response. |
| **Critic / Value Function ($$V(s)$$)** | A separate network estimating expected future reward from state $$s$$. |
| **Reward Model** | A model trained on human preferences to output scalar rewards. |

---

## 1. PPO: Proximal Policy Optimization

PPO is the most mature, battle-tested algorithm in the alignment space. It is an online policy gradient method that optimizes token probabilities while using clipping mechanisms to ensure training stability. 

If a model writes an excellent response but hallucinates at the very end, standard policy gradients (like REINFORCE) punish every token equally. PPO fixes this massive variance through three key innovations:

**1. Advantage Estimation:** Instead of weighting updates by the full trajectory reward, PPO uses the advantage: $$\hat{A}_t = Q(s_t, a_t) - V(s_t)$$. This answers: *"How much better was this specific token than what I'd expect on average?"* **2. Actor-Critic Architecture:** PPO requires four models loaded simultaneously in memory, making it highly computationally expensive:
* Actor (the policy $$\pi_\theta$$ being trained)
* Critic (value network $$V(s)$$)
* Reward Model $$R(s,a)$$
* Reference Model $$\pi_{\text{ref}}$$ (frozen SFT model)

**3. Clipped Surrogate Objective:** This is PPO's defining contribution. It limits how much the policy can change in a single update step to prevent catastrophic collapse. If the probability ratio between the new and old policy is $$r_t(\theta)$$, the objective is:

$$L^{\text{CLIP}}(\theta) = \mathbb{E}\left[\min\left(r_t(\theta)\hat{A}_t,\ \text{clip}(r_t(\theta), 1-\varepsilon, 1+\varepsilon)\hat{A}_t\right)\right]$$

> ##### KL DIVERGENCE: TRUST-REGION VS. DRIFT
> PPO relies on two different KL penalties. **Trust-Region KL** (a forward KL) stabilizes individual update steps against the immediately previous policy. **Drift KL** (a reverse KL) prevents reward hacking by penalizing the policy if it drifts too far from the natural language of the frozen SFT model ($$\pi_{\text{ref}}$$).
{: .block-warning }

---

## 2. DPO: Direct Preference Optimization

PPO is powerful but heavy. DPO completely eliminates the need for both the Reward Model and the Value Function. It operates entirely offline using preference pairs—a "winning" response ($$y_w$$) and a "losing" response ($$y_l$$).

DPO achieves this by mathematically proving that the optimal RL policy can be extracted directly using a Bradley-Terry pairwise preference model. Here is the elegant mathematical trick that makes DPO possible:

<Steps>
{/* Reason: This 4-step mathematical derivation must be read sequentially to understand how the intractable partition function is isolated and ultimately canceled out. */}
  <Step title="1. Define the RLHF Objective" subtitle="Reward and KL Penalty">
    The standard goal is to maximize expected reward while using a KL-divergence penalty ($$\beta$$) to stay close to the frozen reference model:
    $$\max_{\pi} \mathbb{E}_{x, y} \left[ r(x, y) - \beta \log \frac{\pi(y|x)}{\pi_{ref}(y|x)} \right]$$
  </Step>
  <Step title="2. Rewrite in Closed Form" subtitle="The Optimal Policy">
    The mathematically perfect policy ($$\pi^*$$) that solves the objective is the reference policy scaled by the reward. $$Z(x)$$ is a partition function that ensures probabilities sum to 1, but it is intractable (impossible to compute directly).
    $$\pi^*(y|x) = \frac{1}{Z(x)} \pi_{ref}(y|x) \exp\left( \frac{1}{\beta} r(x,y) \right)$$
  </Step>
  <Step title="3. Invert the Equation" subtitle="Solving for the Reward">
    Instead of using a separate neural network to guess the reward, we algebraically solve the equation from Step 2 to define the reward strictly in terms of the policy:
    $$r(x,y) = \beta \log \frac{\pi^*(y|x)}{\pi_{ref}(y|x)} + \beta \log Z(x)$$
  </Step>
  <Step title="4. Cancel Out the Uncomputable Term" subtitle="The Preference Trick">
    We want to maximize the difference between a winning response ($$y_w$$) and a losing one ($$y_l$$). When we subtract $$r(x, y_l)$$from$$r(x, y_w)$$, the uncomputable $$Z(x)$$ terms cancel out entirely!
    $$\mathcal{L}_{DPO} = - \mathbb{E} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)} \right) \right]$$
  </Step>
</Steps>

By optimizing this loss directly, the model increases the log-probability of preferred responses and decreases it for dispreferred ones. Variations like **Iterative DPO** (regenerating preference pairs dynamically to prevent distribution shift) and **KTO** (handling unpaired "thumbs up / thumbs down" data) have made this family of algorithms incredibly popular.

---

## 3. GRPO: Group Relative Policy Optimization

While DPO is highly efficient, PPO's active trial-and-feedback loop is demonstrably better at teaching complex, multi-step reasoning tasks (like math and coding). GRPO offers the ultimate middle ground: the active exploration of PPO, but without the massive memory overhead of the Value Function Critic. This is the algorithm that powered DeepSeek-R1 and V3.

Instead of a Value Function evaluating a single response, GRPO uses a **Group Generator**. 

Given $$G$$sampled responses for the same prompt with rewards$$\{R_1, \ldots, R_G\}$$, the advantage for response $$i$$ is calculated using a simple z-score normalization across the group:

$$A_i = \frac{R_i - \text{mean}(R_1, \ldots, R_G)}{\text{std}(R_1, \ldots, R_G)}$$

By calculating the advantage using group statistics rather than a dedicated Critic network, GRPO dramatically reduces computational costs. 

### The KL Estimator Subtlety
GRPO places the Drift KL penalty *inside* the loss function rather than subtracting it directly from the reward. Recent literature (like *A Comedy of Estimators*, Shah et al. 2026) highlights that GRPO uses the $$K3$$estimator in its loss. While this works beautifully in practice, it is technically biased compared to applying a$$K1$$ estimator directly to the reward—a subtle mathematical quirk that impacts how open-source RL libraries implement the algorithm.

---

## The Evolutionary Arc & Head-to-Head

To summarize how we got here:

$$\text{REINFORCE} \xrightarrow{\text{stability}} \text{PPO} \xrightarrow{\text{simplicity}} \text{DPO} \xrightarrow{\text{efficiency}} \text{GRPO}$$

| Dimension | PPO | DPO | GRPO |
| :--- | :--- | :--- | :--- |
| **Reward Model** | Required (Explicit) | None (Implicit) | Optional (Can use Rules) |
| **Critic Network**| Required | None | None |
| **Models in Memory**| 4 (Actor, Ref, RM, Critic) | 2 (Actor, Ref) | 2–3 |
| **Data Format** | Prompts + RM Scoring | Offline Preference Pairs | Prompts + Group Scoring |
| **Compute Cost** | High | Low | Medium |
| **Best Use Case** | Complex multi-objective alignment | Fast, efficient preference tuning | Math/code reasoning where online rollouts matter |

Each step in this evolution traded a specific type of flexibility for simplicity and efficiency. The right choice depends entirely on your compute budget and whether your task requires deep, online exploration or straightforward style alignment.


```