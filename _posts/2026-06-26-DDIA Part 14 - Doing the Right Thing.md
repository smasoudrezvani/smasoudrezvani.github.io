---
layout: post
title: "Part 14: Doing the Right Thing"
date: 2026-06-26
tags: ddia ethics privacy data-systems surveillance ml-bias
categories: ddia
series: ddia
mermaid:
  enabled: true
  zoomable: true
---

Imagine a physically perfect bridge — flawless steel, mathematically correct load calculations. But the automated toll booth at the entrance systematically refuses entry to certain people, not because they can't pay, but because an opaque software system associated them with a demographic that caused a traffic jam five years ago. The code compiles. The servers run at optimal capacity. On paper, the system is a triumph of engineering. The societal outcome is a disaster. Chapter 14 of DDIA forces a transition from the _how_ of building systems to the _why_.

> ##### Source
>
> Notes drawn from Chapter 14 of _Designing Data-Intensive Applications_ (2nd ed.) by Martin Kleppmann & Chris Riccomini.
> {: .block-tip }

> ##### Created With
>
> These notes were structured with the help of [NotebookLM](https://notebooklm.google.com), using podcast-style audio overviews generated from the book chapters.
> {: .block-tip }

---

## 1. The Illusion of Neutral Technology

Technology is often framed as neutral — a hammer can build a house or break a window; the hammer itself has no moral stance. The chapter challenges this framing for data systems.

When an engineer stares at a database schema, they see strings, integers, and booleans. But those abstractions represent human behaviors, human interests, and human identities. A `Boolean isEligible = false` flag is not neutral:

> That flag might mean a single mother can't get a loan to repair her car, which means she can't get to work, which means she loses her job.

The ACM Code of Ethics has existed for decades and is almost never discussed in engineering practice. Teams optimize for uptime and feature velocity, ignoring the social fallout downstream.

---

## 2. The Engineer's Responsibility

A common defense: "Ethics is the product manager's job. I'm just executing sprint tickets."

The chapter refutes this directly. The software engineer is the one writing the logic that executes business decisions. Every day, they make micro-architectural choices that dictate how a system behaves at scale:

- How to handle edge cases
- What default values to assign
- How to structure the data model
- What data to log and retain

> You cannot evade responsibility by hiding behind "the algorithm made the decision." The algorithm only does what the code — and the data it was trained on — tell it to do.

Unlike a compiler, which throws a deterministic syntax error, ethics require continuous interpretation and dialogue. What one demographic considers a helpful personalized feature, another may consider a privacy violation. This ambiguity makes engineering ethics an ongoing practice, not a one-time compliance checkbox.

---

## 3. Predictive Analytics and the Algorithmic Prison

Predictive analytics in beneficial contexts — weather forecasting, epidemic modeling, traffic routing — are genuinely valuable. They become dangerous when applied to human gatekeeping: predicting criminal recidivism, mortgage default risk, or job performance.

Businesses that deploy these systems are motivated by risk minimization. From their perspective, the cost of a false negative (denying a good loan) is a missed profit opportunity. The cost of a false positive (approving a bad loan) can be severe. So the algorithms are tuned to be conservative — when statistical confidence drops, the system defaults to **No**.

The result is what the text calls the **algorithmic prison**: an individual labeled "high-risk" by an automated system is systematically excluded from employment, housing, and financial services. Unlike the criminal justice system:

- There is no presumption of innocence.
- There is no transparent court hearing.
- There is often no mechanism for appeal.
- The decision arrives as a generic error message.

The exclusion operates in the dark.

---

## 4. Machine Learning as "Money Laundering for Bias"

Machine learning models do exactly one thing: extrapolate patterns from historical training data. If that data contains human biases — and it almost certainly does, because human history is deeply biased — the model doesn't sanitize the bias. It encodes it mathematically into feature weights and amplifies it at unprecedented scale.

> "Machine learning is like money laundering for bias."

When a human makes a biased decision, you can challenge them. You can demand reasoning. You can hold them accountable. When that same human prejudice is passed through the hidden layers of a neural network, it emerges on the other side disguised as **clean, objective, mathematical truth**. People look at the output and say: "The math indicates you are a high-risk candidate, so it must be true." The prejudice has been laundered through an algorithm to appear as empirical science.

---

## 5. Feedback Loops and Systemic Failures

The situation becomes worse when the algorithm's decisions actively alter reality to generate the data that justifies its next decision.

### The Joblessness Spiral

1. An employee contracts a severe illness — a random, uncontrollable event.
2. They miss a month of work, burn through savings, and miss a utility bill. Their credit score drops.
3. On recovery, they apply for a new job. The employer's applicant tracking system uses credit scores as a feature weight. Low score → automatic rejection before a human sees the resume.
4. Unemployed and in financial distress, they miss more bills. Credit score drops further.
5. The algorithm didn't predict failure. It **engineered** it. Its output became its own future input.

### Algorithmic Collusion

In Germany, several gas station networks introduced algorithmic pricing models to stay competitive. Economists later found that the algorithms had independently learned to collude — simultaneously raising prices together to maximize profit, neutralizing market competition with no human instruction. The feedback loop optimized for revenue extraction while ignoring the intended goal of competitive pricing.

---

## 6. Systems Thinking as the Fix

The solution is not a better unit test. A unit test verifies that a function returns the correct integer. It says nothing about whether the system, interacting with real humans in the real world, produces a just outcome.

**Systems thinking** is a holistic mindset: map the entire environment. Evaluate the physical placement of sensors, the human behaviors of people interacting with the system, and how the system's output physically alters the inputs that feed back into it.

An engineer writing predictive logic must ask:

- How does the output of my code affect the human on the other end?
- How does that person's subsequent behavior feed back into my training data?
- Where are the self-reinforcing feedback loops?
- What are the second-order effects I'm not seeing in a sterile staging environment?

---

## 7. Surveillance Capitalism

Training predictive algorithms requires data. Vast amounts of data. The chapter examines how tech platforms acquire it.

A fundamental shift occurred over the past two decades:

| Model                                              | Who is the customer?                                                   |
| -------------------------------------------------- | ---------------------------------------------------------------------- |
| Traditional software (spreadsheet, word processor) | **The user** — you exchange data for utility                           |
| Surveillance advertising (social media, free apps) | **The advertiser** — your behavioral profile is the product being sold |

When your behavior is tracked silently as a side effect of using a service — where your mouse hovers, how long you linger on a photo, your scroll velocity — that telemetry is aggregated to sell targeted advertising. You are not the customer. You are the product.

### Daniel J. Bernstein's Substitution Test

Replace the word "data" with "surveillance" in standard corporate language:

> _"In our surveillance-driven organization, we collect real-time surveillance streams and store them in our surveillance warehouse. Our surveillance scientists derive insights to manipulate the user experience."_

The substitution is jarring — but the authors argue it is the most accurate description of the business model.

---

## 8. Privacy as a Decision Right

The common defense — "I have nothing to hide, let them track me" — misunderstands what privacy is.

> **Privacy is not about secrecy. It is the fundamental human autonomy to choose what information you reveal, to whom, and in what context.**

Mass surveillance strips away that decision right. The corporation determines what is extracted and who has access.

GDPR mandates "freely given" consent. But if every employer requires a LinkedIn profile and every peer group organizes exclusively on a specific social platform, clicking "I agree to be tracked" is not a free choice. The social and economic friction of opting out is engineered to be prohibitively high. That is **coercion by monopoly**.

### The Secondary Data Market Problem

The chapter provides a sobering example: car insurance companies now purchase telemetry data from vehicle manufacturers — acceleration patterns, braking habits, GPS location — to set premiums, often without the driver's explicit knowledge or consent.

Sensor accuracy has advanced to the point where security researchers demonstrated that the accelerometer in a standard smartwatch (designed to count steps) can deduce passwords being typed on a physical keyboard with alarming accuracy. The seemingly innocuous data from one sensor can be repurposed for total behavioral capture.

---

## 9. Data as Uranium, Not Gold

The tech industry's default frame: **data is the new oil** (or gold) — a resource to be mined and refined for value.

The chapter proposes a corrective: **data is the new uranium**. Uranium is highly valuable in the right context but is inherently hazardous to store. Hoarding gold is inert. Hoarding a mountain of sensitive human surveillance data is dangerous:

- It can be breached by hackers.
- It can be compromised by foreign intelligence services.
- If the company goes bankrupt, the database is a **legally transferable corporate asset** sold in bankruptcy court — violating the original consent of every user whose data is in it.

> **Geopolitical implication**: today's friendly democratic government may not be in charge of a jurisdiction in 10, 20, or 50 years. Building a turnkey mass surveillance infrastructure to optimize ad click-through rates means you have zero control over who eventually holds the cryptographic keys.

---

## 10. The Industrial Revolution Analogy

The Industrial Revolution brought extraordinary economic growth and raised living standards — but also unchecked horrors: toxic smog, chemical waste in drinking water, child labor in dangerous conditions. Decades of severe human suffering passed before society instituted environmental protection laws, workplace safety regulations, and child labor prohibitions. Factory owners complained bitterly that these regulations would ruin profit margins and stifle innovation. Society as a whole benefited enormously.

Bruce Schneier's formulation:

> **"Data is the pollution problem of the information age."**

Future generations will look back at the unregulated early decades of the information age the way we look back at Victorian factory owners dumping chemical waste into drinking water.

---

## 11. Data Minimization vs. Data Maximization

The chapter identifies a fundamental clash between two philosophies:

| Philosophy                       | Principle                                                                                                                        | Driven by                                  |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| **GDPR / Data minimization**     | Collect only the minimum data strictly necessary for the stated service. Delete it immediately when the transaction is complete. | Privacy regulation, human rights           |
| **Big data / Data maximization** | Collect everything you can. Never delete anything. Future ML models might extract value from it.                                 | Revenue optimization, analytical potential |

These are not minor policy differences. They are contradictory architectural philosophies. An engineer building a new feature is forced to choose, often implicitly, which philosophy their code embodies.

The chapter's argument: **data you choose not to collect cannot be stolen, leaked, or weaponized against your users**. Data minimization is not just a regulatory compliance exercise. It is the most effective security control available.

---

## Summary

| Topic                     | Core Insight                                                                                               |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Neutral technology        | Schemas contain human dignity; ignoring downstream harm is not neutrality                                  |
| Engineer's responsibility | You write the logic that executes the decision; "the algorithm decided" is not an absolution               |
| Predictive analytics      | Systems tuned to minimize false positives systematically exclude at scale                                  |
| Bias laundering           | ML encodes historical prejudice as mathematical truth, removing accountability                             |
| Feedback loops            | An algorithm's decisions can alter reality in ways that feed back to justify the next decision             |
| Systems thinking          | Map the entire human-system interaction, not just the function's return value                              |
| Surveillance capitalism   | When the product is "free," the user is the product; behavioral tracking inverts the customer relationship |
| Privacy as autonomy       | Privacy is the right to choose what you reveal; coercion by monopoly negates free consent                  |
| Data as uranium           | Sensitive data is hazardous to store; breach, government seizure, and bankruptcy all threaten it           |
| Data minimization         | Not collecting data is the strongest possible protection against its misuse                                |

The mathematical logic of your code may be perfectly clean. If the automated tollbooth on your beautifully engineered bridge systematically locks people out of society, the bridge is broken. The engineer has both the agency and the responsibility to fix it.
