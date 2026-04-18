---
layout: post
title: "Beyond Correlation: A Guide to Causal Inference and Impact Evaluation"
date: 2026-03-31 12:00:00
description: "Navigating the Potential Outcomes Framework to extract true causal signals from observational data."
tags: data-science machine-learning causal-inference impact-evaluation
categories: ai-research
chart:
  echarts: true
mermaid:
  enabled: true
  zoomable: true
---

In machine learning, we spend most of our time optimizing for pure prediction. We feed thousands of features into complex models to minimize a loss function. But when it comes to business strategy, public policy, or algorithmic trading, predicting an outcome isn't enough; we need to know what *caused* it.

We need to answer "what if" questions: What if we deployed this new LLM optimizer? What if we applied this trading tax? To do this, we must navigate the Potential Outcomes Framework, where every individual has a potential outcome if treated ($$Y_{1i}$$) and if untreated ($$Y_{0i}$$). Because we can only ever observe one of these realities, calculating the true Average Treatment Effect (ATE) requires defeating selection bias.

Here is a technical breakdown of the core methods used to extract true causal signals from observational data.

---

### 1. Instrumental Variables (IV): The Exogenous Nudge

When treatment adoption is completely voluntary (endogenous), standard OLS regressions suffer from "selection on unobservables." IV solves this by finding an external variable (the instrument, $$Z_i$$) that nudges people toward the treatment without directly affecting the outcome.

**The Intuition:** Imagine offering a random subset of algorithmic traders a 50% discount on a new latency-reducing API. The discount itself doesn't make their trading algorithms better, but it encourages them to buy the API, which might.


<div class="d-flex justify-content-center" markdown="1">

```mermaid
graph LR
    Z((Z: Discount<br>Instrument)) -->|Relevance| D((D: API Usage<br>Treatment))
    D --> Y((Y: Trading Profit<br>Outcome))
    U((U: Trader Skill<br>Unobserved)) -.-> D
    U -.-> Y
    Z -.->|Exclusion Restriction<br>No Direct Path| Y
    
    classDef default fill:#1a1a1a,stroke:#888,stroke-width:2px,color:#fff;
    classDef unobserved fill:#331111,stroke:#d32f2f,stroke-width:2px,color:#fff;
    class U unobserved;
````

\</div\>

**The Math:** The IV estimator isolates the variance in the treatment ($$D_i$$) driven purely by the instrument ($$Z_i$$). For a binary instrument, this simplifies to the Wald Estimator: 

$$ \frac{E[Y_i|Z_i=1] - E[Y_i|Z_i=0]}{E[D_i|Z_i=1] - E[D_i|Z_i=0]} $$

> ##### KEY ASSUMPTIONS
> 1. **Relevance:** The instrument must actually change treatment probability ($$Cov(Z_i, D_i) \neq 0$$).
> 2. **Exclusion Restriction:** The instrument only affects the outcome through the treatment ($$E[u_i | Z_i, X_i] = 0$$).
> 3. **Monotonicity:** The instrument pushes everyone in the same direction, ruling out "defiers". Under these rules, IV estimates the LATE (Local Average Treatment Effect) specifically for the "compliers".
{: .block-warning }

---

### 2. Panel Data & Fixed Effects: Erasing the Unobservables

When tracking the same entities over multiple time periods, we can eliminate omitted variable bias caused by unobserved factors that are completely constant over time (like a company's innate corporate culture or a region's fixed geography).

**The Intuition:** By mathematically subtracting an entity's historical average from its current state, any baseline characteristic that doesn't change over time is perfectly destroyed, leaving only the variation we care about.

**The Math:** The "Entity-Demeaned" OLS estimator:

$$ (Y_{it} - \overline{Y}_i) = \beta_1(X_{it} - \overline{X}_i) + (u_{it} - \overline{u}_i) $$

The unobserved fixed effect $$\alpha_i$$ is eliminated because $$\alpha_i - \alpha_i = 0$$.

> ##### KEY ASSUMPTIONS
> **Strict Exogeneity** ($$E[u_{it} | X_{i1},...,X_{iT}, \alpha_i] = 0$$), which explicitly rules out feedback loops where today's shock affects tomorrow's treatment. *Note: You must use clustered standard errors to account for serial correlation within entities.*
{: .block-warning }

---

### 3. Difference-in-Differences (DiD): The Parallel Counterfactual

When a policy or intervention hits one group but not another, DiD calculates the causal effect by analyzing the change in outcomes over time across the two groups.

**The Intuition:** If Amsterdam introduces a new tech hiring subsidy but Rotterdam does not, we don't just compare the two cities (they are inherently different). Instead, we see how much Amsterdam's employment changed, and subtract the natural baseline change observed in Rotterdam over the exact same period.

*Interact with the chart below to see how DiD isolates the treatment effect from the counterfactual trend:*


```echarts
{
  "title": { "text": "Difference-in-Differences (DiD)", "left": "center" },
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["Amsterdam (Treated)", "Rotterdam (Control)", "Counterfactual"], "bottom": 0 },
  "xAxis": { "type": "category", "data": ["Pre-Treatment", "Post-Treatment"] },
  "yAxis": { "type": "value", "min": 0, "max": 100, "name": "Employment" },
  "series": [
    { "name": "Amsterdam (Treated)", "type": "line", "data": [40, 85], "lineStyle": {"width": 3, "color": "#FF5722"}, "symbolSize": 8 },
    { "name": "Rotterdam (Control)", "type": "line", "data": [30, 50], "lineStyle": {"width": 3, "color": "#2196F3"}, "symbolSize": 8 },
    { "name": "Counterfactual", "type": "line", "data": [40, 60], "lineStyle": {"type": "dashed", "width": 2, "color": "#FF5722"}, "itemStyle": {"color": "#FF5722"} }
  ]
}
```

**The Math:** Repeated cross-sectional formulation:

$$ Y_{it} = \beta_0 + \beta_1 X_{it} + \beta_2 D_i + \beta_3 S_t + u_{it} $$

Here, the interaction term $$X_{it} = (D_i \times S_t)$$ is the DiD estimator ($$\beta_1$$), perfectly isolating the treatment effect.

> ##### KEY ASSUMPTIONS
> **Parallel Trends.** In the absence of the treatment, the treatment group and control group would have followed the exact same trajectory.
{: .block-warning }

---

### 4. Regression Discontinuity Design (RDD): The Arbitrary Cutoff

RDD is perhaps the most elegant trick in causal inference. It exploits strict, arbitrary rules in the real world to mimic a randomized control trial locally.

**The Intuition:** Imagine an algorithmic trading system that executes a massive buy order only if a stock's RSI drops below exactly 30.0. A stock with an RSI of 29.9 and a stock with 30.1 are fundamentally identical, driven apart by pure random market noise. Therefore, treatment assignment right at the boundary is "as if" random.

```echarts
{
  "title": { "text": "Sharp Regression Discontinuity", "left": "center" },
  "xAxis": { "type": "value", "min": 20, "max": 40, "name": "Running Variable (e.g., RSI)", "nameLocation": "middle", "nameGap": 30 },
  "yAxis": { "type": "value", "min": 0, "max": 100, "name": "Outcome" },
  "series": [
    { "name": "Untreated", "type": "scatter", "data": [[22, 25], [25, 30], [27, 35], [28, 38], [29.5, 42]], "itemStyle": {"color": "#2196F3"} },
    { "name": "Untreated Trend", "type": "line", "data": [[20, 20], [30, 45]], "lineStyle": {"color": "#2196F3", "width": 2}, "symbol": "none" },
    { "name": "Treated", "type": "scatter", "data": [[30.5, 75], [32, 78], [35, 85], [38, 92], [39, 95]], "itemStyle": {"color": "#FF5722"} },
    { "name": "Treated Trend", "type": "line", "data": [[30, 72], [40, 98]], "lineStyle": {"color": "#FF5722", "width": 2}, "symbol": "none",
      "markLine": {
        "data": [ { "xAxis": 30, "label": {"formatter": "Cutoff\n(RSI = 30)", "position": "middle"} } ],
        "lineStyle": { "color": "#fff", "type": "dashed", "width": 2 }
      }
    }
  ]
}
```

**The Math:** For a Sharp RDD allowing different slopes on either side of the cutoff $$\overline{S}$$:

$$ Y_i = \alpha + \beta(S_i - \overline{S}) + \rho D_i + \gamma D_i(S_i - \overline{S}) + u_i $$

The parameter $$\rho$$ captures the exact vertical "jump" at the threshold, which is the causal effect.

> ##### KEY ASSUMPTIONS
> **Continuity.** The relationship between the running variable ($$S_i$$) and the outcome must be perfectly smooth in the absence of treatment, and agents cannot perfectly manipulate their score to cross the boundary.
{: .block-warning }

---

### 5. Synthetic Control Method (SCM): The Frankenstein Match

When a massive shock affects exactly one unit (e.g., one country passes a unique AI law), DiD fails because no single other country is a perfect parallel match.

**The Intuition:** SCM uses an optimization algorithm to build a "synthetic" version of the treated unit by calculating a weighted average of untreated units in the donor pool.

**The Math:** It minimizes the distance between the treated unit's pre-treatment covariates ($$X_1$$) and the donor pool's covariates ($$X_0$$):

$$ W^* = \arg \min || X_1 - X_0 W || $$

> ##### KEY ASSUMPTIONS
> The weights must sum to 1 to prevent wild extrapolation. Because large-sample asymptotics fail with $$N=1$$, inference relies on **Placebo Tests** (applying the algorithm to the untreated units to see if the real effect is an outlier).
{: .block-warning }

---

### 6. Double Machine Learning (DML): ML Meets Causality

When dealing with a high-dimensional feature space (thousands of user covariates), standard OLS collapses. But if we try to use machine learning (like Random Forests) to control for them, the model's built-in regularization penalty shrinks the causal estimate toward zero, creating regularization bias.

**The Intuition:** DML uses ML twice—once to predict the outcome, and once to predict the treatment—isolating the residuals.

> ##### KEY ASSUMPTIONS
> It relies on **Neyman Orthogonality**, setting up a mathematical score function that is completely insensitive to the small prediction errors (regularization biases) made by the nuisance ML estimators. To prevent overfitting, DML rigorously enforces Sample Splitting (Cross-Fitting).
{: .block-warning }

---

### 7. Optimal Policy Learning: From Insights to Decisions

Finally, DML allows us to calculate the Conditional Average Treatment Effect (CATE)—how a treatment affects highly specific sub-groups. We use this to build mathematical allocation rules ($$\pi(X_i) \in \{0, 1\}$$).

**The Math:** The algorithm searches for the policy tree that maximizes the total benefit across the population:

$$ \pi^*(X_i) = \arg\max E[(2\pi(X_i) - 1) \times CATE(X_i)] $$

**The Intuition:** It creates a mathematical seesaw. If a sub-group's CATE is positive, multiplying it by $$+1$$ increases the total score, so the algorithm assigns the treatment. If the CATE is negative, assigning the treatment would penalize the score, so the algorithm denies it.

---

> Causal inference forces us to stop asking "What pattern exists in the data?" and start asking **"What mechanism generated the data?"**