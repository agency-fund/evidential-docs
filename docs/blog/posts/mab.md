---
title: Multi-Armed & Contextual Bandits Experiments
date: 2025-08-20
tags:
  - bandits
  - experiments_types
---

Run adaptive experiments, learn on the fly, send more traffic to the best options — and even personalize choices using context.
<!-- more -->

# 🎰 Multi-Armed & Contextual Bandits in Evidential

We’re excited to announce that **Evidential now supports Multi-Armed Bandit and Contextual Bandit experiments** — alongside our frequentist A/B tests.

---

## What are Bandits?

Think of each version of your intervention as "arm" in a slot machine. Your "reward" when you pull the arm is the outcome you observe when you deploy your intervention to your beneficiaries.

A **multi-armed bandit** algorithm learns which arm pays off best and automatically directs more participants toward it -- so you optimize your interventions on the go, rather than at the end of your experiment.

A **contextual bandit** goes one step further: it takes into account **information about participants** — such as demographics or behavior — to decide which arm is likely to work best for *that specific user*.

---

## Why this matters

- Traditional A/B tests split participants evenly across variations, wasting time and resources on poor options.  
- **Multi-armed bandits** quickly identify the best variation and shift traffic toward it.  
- **Contextual bandits** personalize in real time, matching people with the intervention most likely to help them.  

Together, these approaches reduce the audience size and time needed to discover what works, **maximizing impact while still maintaining experimental rigor**.

---

## How to use it

It’s just as straightforward as creating any other experiment in Evidential:  
when you set up an experiment, simply choose **Multi-Armed Bandit** or **Contextual Bandit** in the experiment type dropdown. Evidential takes care of the complex math under the hood — no extra coding required.

---

## What problems can this solve?

- Optimizing messaging, timing, or program variations in real time.  
- Testing multiple strategies when outcomes are scarce or expensive to measure.  
- Personalizing interventions based on participant characteristics or circumstances.  

---

✨ With bandits, you don’t just *test* — you **learn and improve continuously while running your experiment**.

---

*Released: August 20, 2025*
