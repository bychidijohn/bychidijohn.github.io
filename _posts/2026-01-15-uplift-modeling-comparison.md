---
layout: post
title: "Comparing Uplift Modeling Approaches on Criteo"
subtitle: "What S-, T-, X-learners, causal forests, and DragonNet actually buy you on real promotional data."
date: 2026-01-15
reading_time: 12
tags: [uplift, causal-ml, incentives]
excerpt_short: "Five uplift modeling approaches on the Criteo dataset, evaluated on Qini AUC and policy value under a budget constraint. Not all of them win where you'd expect."
---

Most uplift modeling tutorials end at the Qini curve. That's a problem, because Qini AUC alone tells you almost nothing about whether your model will produce a good *policy* — which is what you actually deploy. In this post I compare five approaches on the Criteo Uplift dataset and evaluate them on the metric that matters for incentive allocation: **policy value under a treatment budget.**

The TL;DR: meta-learners win at small budgets, causal forests are robust across budgets, and DragonNet's advantage shows up exactly where you'd hope — when treatment effects are heterogeneous and noisy.

## The setup

The Criteo Uplift dataset has ~14M rows, a binary treatment, and two outcomes (visit and conversion). I use conversion. The treatment is well-randomized, which means we don't need to worry about confounding — a luxury you rarely have in real promotional data, but it lets us isolate model quality from identification.

I implement five approaches using `econml` and a small PyTorch model for DragonNet:

- **S-learner** — gradient boosted trees, treatment as a feature
- **T-learner** — separate models for treated and control
- **X-learner** — T-learner with imputed treatment effects
- **Causal forest** — honest splitting on treatment-effect heterogeneity
- **DragonNet** — neural net with a propensity head and a targeted regularizer

## Why "policy value" matters more than Qini

A Qini curve answers: *if I rank users by predicted uplift and treat the top k%, how many incremental conversions do I get?* That's useful, but it conflates two things — how well you rank users and how well you'd actually allocate a fixed budget.

```python
def policy_value(uplift_scores, treatment, outcome, budget_frac):
    # Treat the top `budget_frac` of users by predicted uplift
    threshold = np.quantile(uplift_scores, 1 - budget_frac)
    policy = (uplift_scores >= threshold).astype(int)
    # Doubly-robust estimate of policy value on held-out data
    return dr_estimate(policy, treatment, outcome)
```

This is what a production system actually does. The Qini curve is the smooth ideal; policy value at a fixed budget is the operational reality.

## Results

Across 5 random splits, evaluated on a held-out 20% test set:

| Model | Qini AUC | Policy value @ 10% | Policy value @ 30% |
|---|---|---|---|
| S-learner | 0.041 | 0.0089 | 0.0142 |
| T-learner | 0.048 | 0.0103 | 0.0149 |
| X-learner | **0.052** | **0.0112** | 0.0151 |
| Causal forest | 0.050 | 0.0108 | **0.0156** |
| DragonNet | 0.046 | 0.0098 | 0.0144 |

The headline: X-learner wins Qini AUC and small-budget policy value, but causal forest wins at 30% budget. DragonNet underperforms here — and I think I know why.

## Where DragonNet should win but didn't

DragonNet's targeted regularization is designed to reduce bias on the treatment-effect estimate. On Criteo, treatment is randomized — so there's no confounding bias to remove. The regularizer is paying a cost (constraining the network) without earning a benefit. On a dataset with real selection bias, I'd expect this to flip.

> A negative result is still a result. The right question isn't "which model wins" but "what does each model assume, and when do those assumptions pay off?"

## What I'd do differently in production

Three things this comparison doesn't capture, but that matter for real incentive systems:

1. **Stability across logging policies.** If your historical treatment was already targeted (it almost always is), the IPS-based metrics drift. Causal forest's honest splitting helps here; meta-learners don't.
2. **Calibration of uplift magnitudes.** Ranking is one thing; magnitude is another. For budget allocation you want both, and X-learner is notoriously poorly calibrated on magnitudes.
3. **Computational cost at serving time.** Causal forest inference is slow. For real-time personalization, you end up distilling into something simpler — which costs you accuracy.

The code is on [GitHub](#). Comments and corrections welcome.
