# A/B Testing Data Analysis

This page answers the most common questions about how Evidential handles analytics for A/B testing, outlining the statistical logic and assumptions that power these experiments in Evidential’s experiment engine.

______________________________________________________________________

## Power Calculations

We calculate power using a **two-sample t-test**, with a default, editable **80% power (1−β)** and **0.05 significance level (α)**.
We derive the baseline value for the outcome from existing potential participants and compute a default target effect size of:

> Target Effect Size = Baseline × 0.1

______________________________________________________________________

## Assignments

### Pre-assigned

Static, **stratified randomized assignment** by selected strata.

### Online

Dynamic **randomized assignment** without any stratification. Higher risk of statistical imbalance in smaller samples.

______________________________________________________________________

## Balance Checks

Omnibus test predicting treatment assignment using any available user-level data variables.
This can be any combination of outcomes, strata, or filter values.
We should **not** be able to predict treatment assignment with any individual or combination of these variables.

We report the **F-statistic test of joint significance** in a standard OLS regression.

______________________________________________________________________

## Strata

Also known as **covariates** or **controls**.
These are user-level characteristics that are considered **prior to treatment assignment**.
They are used primarily to stratify treatment assignment but can also be used to:

- Estimate **heterogeneous treatment effects**, and
- Achieve (marginally) improved **efficiency in estimating standard errors**.

______________________________________________________________________

## Data Warehouse Metrics

Outcome metrics that are the **target of experiments** are stored in a data warehouse table at a **user level**.

These metrics should be the **primary metrics** that the organization holds itself accountable to.
They should match the metrics the organization already uses to assess its programs’ operational and impact performance.

______________________________________________________________________

## Regression

We estimate the **intent-to-treat average treatment effect (ATE)** using the following model:

> Metricᵢ = β₀ + β₁ × Treatmentᵢ + Controlsᵢ + εᵢ

Controls include pre-specified strata as well as pre-treatment values for the outcome metric.

______________________________________________________________________

## Interpretation of Results

We run a **linear OLS model**, where we interpret the coefficient on the treatment indicator (**β₁**) as the:

> Intent-to-treat, average treatment effect of the treatment on the chosen metric.

______________________________________________________________________

*Evidential handles all the above automatically — so you can focus on learning what works, not on the math behind it.*
