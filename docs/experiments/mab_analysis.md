# Multi-Armed Bandits Data Analysis

This page answers the most common questions about how Evidential handles analytics for Multi-Armed Bandits (MABs), outlining the statistical logic and assumptions underlying our MAB implementation.

______________________________________________________________________

## MAB Algorithm

Let's imagine that you have set up variants of an experiment, and for each variant you have some prior probability of a desired result. You serve your each of your users one of these variants (the strategy for choosing the variant is based on the prior probabilities), and observe the result of their interaction with it: for example, if you are experimenting with a website interface, you might want to track when a user returns to a page, clicks on a link, etc.

Once you have observed the result, the algorithm updates your arm / variant's probability of achieving the desired result. The next time you serve a user one of the variants, the experiments engine uses these updated probabilities to determine which variant to show them.

Since we update the probabilities for the variants with every result observation, at any given time, you can observe the updated probability of success for every arm. The best-performing variant at the end of the experiment is the one with the highest probability. For example, in the plot below variant 2 is the best-performing one at the end of 200 observations

<img src="../images/mab_arms.png"/>

______________________________________________________________________

## Allowed Priors and Outcome Distributions

In our current implementation of MABs, we support the following configurations of priors and outcome distributions:

| Prior    | Outcome                             |
| -------- | ----------------------------------- |
| Beta     | Binary (Bernoulli distribution)     |
| Gaussian | Real-valued (Gaussian distribution) |

______________________________________________________________________

## Updating the Probabilities of Success for Each Arm

We use Thompson sampling to update the probabilities of success for each arm. The exact update procedure depends on the prior and outcome distribution you choose for your experiment. This allows us to balance exploration and exploitation: we serve the current best-performing variant more often, but we also continue to explore the other variants to ensure that we are not missing out on a better-performing one because of temporary fluctuations in the observed results.

??? note "Show me the math!"

    ??? note "Beta-binomial bandits"

        Given $N$ variants of a feature or implementation, if we assume that the observed result $y$ of these experiments is binary-valued, we can model the result likelihood as follows:
        $$

        \begin{equation}
        y \sim \text{Bernoulli} (\theta_i)
        \end{equation}

        $$
        where $\theta_i$ denotes the binomial probability of the outcome $y$ under variant $i (= 1, \ldots .N)$

        We also assume Beta priors for $\theta_i$:
        $$

        \begin{equation}
        \theta_i \sim \text{Beta} (\alpha_i, \beta_i)
        \end{equation}

        $$
        We require users to set $\\alpha_i$ and $\\beta_i$ parameters for each variant, _before_ the experiment begins. The mean of the distribution is $\\frac{\\alpha_i}{\\alpha_i + \\beta_i}$ -- this means that a larger value of $\\alpha\_{\\cdot}$ corresponds to a higher probability of success, and $\\beta\_{\\cdot}$ is inversely proportional to the probability of success.


        During the course of the experiment, we choose the variant $j$ to present to the user using Thompson sampling:
        $$
        \begin{equation}
        j = \text{argmax}\ \[ \theta_i \]^N_{i = 1}
        \end{equation}
        $$

        Finally, once we have observed an outcome $y$ for a given variant, we can obtain the posterior distribution for that variant. Since the beta and binomial distributions are conjugate, this is a relatively straightfoward update: i.e. given an observation $y$,
        $$
        \begin{equation}
        \theta^{(\text{post})}_j \sim \text{Beta} (\alpha_j + y, \beta_j + (1 - y))
        \end{equation}
        $$
        This then becomes the new prior from which we sample variants while choosing arms / variants.

    ??? note "Gaussian priors and real-valued outcomes"

        With real-valued outcomes and Gaussian priors, the likelihood becomes:
        $$
        \begin{equation}
        y \sim \mathcal{N} (\theta_i, \sigma^2)
        \end{equation}
        $$

        In our current implementation we set $\sigma = 1$. The corresponding priors for each variant are:
        $$
        \begin{equation}
        \theta_i \sim \mathcal{N} (\mu_i, \sigma^2_i)
        \end{equation}
        $$

        Thompson sampling proceeds similarly to the Beta-binomial case:
        $$
        \begin{equation}
        j = \text{argmax}\ \[ \theta_i \]^N_{i = 1},
        \end{equation}
        $$
        and the posterior update given an observation $y$ is as follows:
        $$
        {\sigma }^{(\text{post})}_j = \frac{1}{\sqrt{\frac{1}{\sigma^2_j} + \frac{1}{\sigma^2}}}
        $$
        $$
        \mu^{(\text{post})}_j = \frac{\frac{\mu_j}{\sigma_j^2} + \frac{y}{\sigma^2}}{\frac{1}{\sigma^2_j} + \frac{1}{\sigma^2}}
        $$
        $$
        \begin{equation}
        \theta^{(\text{post})}_j \sim N (\mu^{(\text{post})}_j, (\sigma^{(\text{post})}_j)^2)
        \end{equation}
        $$

    ### Additional Resources
    You can learn more about multi-armed bandits from the following resources:

    1. [Multi-armed Bandits with Thompson Sampling](https://www.youtube.com/watch?v=TdjOAfk7iVA)

    2. [Reinforcement Learning: An Introduction, Sutton and Barto, Chapter 2](http://www.incompleteideas.net/book/RLbook2020.pdf)


______________________________________________________________________

*Evidential handles all the above automatically — so you can focus on learning what works, not on the math behind it.*
