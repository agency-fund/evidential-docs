# Experiments

Evidential supports different types of experimentation, allowing you to test different variations of your digital intervention and compare their performance.

This can help you identify which features or approaches are most effective in achieving your desired outcomes, and make data-driven decisions about how to improve your program.

# Types of Experiments

## A/B Testing

The most common type of experiment, where users are randomly assigned to one of two (or more) groups, each receiving a different version of the intervention. This allows you to compare the performance of different variations and identify which one is more effective.

## Multi-armed Bandits

Multi-armed Bandits (MABs) are useful for running experiments where you have multiple variants of a feature / implementation that you want to test, and want to automatically converge to the variant that produces the best results. MABs use an algorithm to allocate more traffic to the better-performing variants over time, allowing you to optimize your experiment while it is running.

## Contextual Bandits

Contextual bandits (CMABs), similarly to multi-armed bandits (MABs), are useful for running experiments where you have multiple variants of a feature / implementation that you want to test. However, the key difference is that contextual bandits take information about the end-user (e.g. gender, age, engagement history) into account while converging to the best-performing variant.

# Online vs Offline Experiments

Evidential supports both online and offline experiments, allowing you to choose the approach that best fits your needs and resources.

A/B experiments can be run both online and offline, while multi-armed and contextual bandits are (currently) only available as online experiments.

**Online experiments** are run in real-time, with users being randomly assigned to different groups as they interact with the intervention. This allows you to quickly gather data and make adjustments to your experiment as needed.

**Offline experiments**, on the other hand, are run on pre-existing data. This can be useful if you want to test on a pre-specified cohort of users, or if you simply want to run analysis on historical data.
