---
layout: post
title: "The 3.5-Point Win That Disappeared"
date: 2026-10-26 12:00:00 +0800
description: "Why an optimized data mixture can win at a small training budget, lose its lead as training continues, and retain an advantage when there is a correctable train–target gap."
published: false
permalink: /drafts/part-1-budget-first/
lang: en
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Local alternative draft for Part 1. Keep unpublished until reviewed.
Based on Advancement or Acceleration? How Data Curation Improves LLM Training.
Paper: https://arxiv.org/abs/2610.36983. No change to the existing post or translation. -->

> **Designing Training Data for Language Models · Part 1 of 3**<br>
> *When to optimize, what makes data good, and how to change it.*<br>
> **Advancement vs. Acceleration** · [What Makes Training Data Good?]({{ '/blog/2026/what-makes-data-good/' | relative_url }}) · [From Target to Operator]({{ '/blog/2026/from-target-to-operator/' | relative_url }})

We were trying to improve a language model by optimizing its training-data mixture. At first, it seemed to work.

Following the setup from [Aioli](https://arxiv.org/abs/2411.05735), we trained a 160M-parameter model on seven [SlimPajama](https://www.cerebras.net/blog/slimpajama-a-627b-token-cleaned-and-deduplicated-version-of-redpajama) domains. We used [RegMix](https://arxiv.org/abs/2407.01492) to search for a promising mixture, then trained a fresh model with it. Our baseline sampled the seven domains uniformly. Evaluation also gave each domain equal weight.

At 2,000 steps, the optimized mixture was **3.5 averaged-perplexity points better**. At that budget, the comparison was convincing.

Then we continued training both models, consuming more data under their respective mixtures. The baseline caught up. By the final checkpoint, the optimized mixture had lost its lead.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*The difference is RegMix minus Uniform, so lower is better. RegMix leads by 3.5 perplexity points at 2,000 steps and finishes slightly worse. Averaged perplexity is the exponential of the mean held-out loss across the seven domains.*

This is the pattern I want to understand: **data optimization can help at a small training budget, yet lose its advantage as that budget grows. Why? And when should we expect the advantage to survive?**

Here, a larger budget means more tokens consumed during training with the model size fixed. The size of the available corpus is a separate variable, which will matter later.

## What did the early win actually buy us?

At a fixed budget, a lower loss tells us that one recipe has produced a better model *so far*. It leaves open two possibilities:

> **Acceleration:** reaching a given performance level with less data.<br>
> **Advancement:** improving the best target performance attainable in the large-data limit.

The early RegMix result is consistent with acceleration: a better allocation helps while the baseline is still learning. A single checkpoint cannot tell us whether it also changes the endpoint.

There is a straightforward reason allocation can matter more at small budgets. Different domains can require different amounts of data to learn. Spending more of a limited budget on a harder domain can lower the average target loss sooner. As the baseline receives more examples, it may learn what the optimized mixture helped the other model learn earlier.

Acceleration is valuable. If a recipe reaches the required performance with substantially less compute, it has delivered a useful result. But that result alone does not establish a better endpoint. We need to ask what happens when the baseline gets more data too.

The distinction becomes more interesting once we look at cases where the baseline *does not* catch up within the tested budget.

## When the advantage survives

We compared Uniform with [RegMix](https://arxiv.org/abs/2407.01492), [DoGE](https://arxiv.org/abs/2310.15393), [Skill-It](https://arxiv.org/abs/2307.14430), [Aioli](https://arxiv.org/abs/2411.05735), and [TANDEM](https://openreview.net/forum?id=szBFUtBzWP) across three settings. The 160M model and training protocol stayed the same; we changed the available corpus or the target. Each experiment was repeated five times.

**First, the approximately matched setting from the opening.** Training and evaluation weighted the same seven domains equally, and the pool was large enough to avoid repeatedly sampling depleted domains. The final results showed no reliable improvement over Uniform: for example, Uniform reached 19.71 ± 0.17 averaged perplexity, RegMix 19.80 ± 0.12, and TANDEM 19.66 ± 0.15. These are means and standard deviations across runs.

**Second, a shifted target.** We held out CommonCrawl for evaluation and trained on the other six domains. Uniform exposure across those domains no longer matched the target. Related material, such as C4, still provided a way to improve alignment. Optimized mixtures reduced target perplexity from 49.19 to roughly 40–41, retaining an advantage through the largest tested budget.

**Third, a restricted corpus.** We kept the uniformly weighted target but reduced the candidate pool to 300M tokens, preserving its original domain proportions. Small domains ran out early and were sampled repeatedly. More training then meant more repetition, with overfitting in those domains. Reallocation helped by reducing that exposure: TANDEM improved final averaged perplexity from 28.60 to 26.02.

{% include figure.liquid path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Final gains over Uniform under the three settings. The contrast is between tasks: the same method family has little final headroom in the matched setting and substantial gains in settings with a correctable gap.*

The broad contrast also held when we repeated the mixture comparison between Uniform and TANDEM at 1.1B parameters. The matched-setting gain remained small; the shifted-target and restricted-corpus gains remained substantial.

These experiments suggest a more useful question than whether data optimization “works at scale”: **what is the baseline failing to learn from the exposure it receives, and can a different allocation fix that?**

## A gap gives reallocation something to correct

In the opening experiment, Uniform was doing something important: its population training objective already matched the target objective. Both assigned equal importance to the same domains, with matched distributions within each domain.

Imagine the idealized endpoint of that process. With enough independent data and a global solution of the training problem, the baseline already minimizes the target loss within the fixed model class. Another mixture can help it get there sooner, but cannot produce a model with an even lower value of that same objective.

This is the simple obstruction behind the disappearing win. It also explains why domain difficulty alone need not create a better endpoint. Harder domains can require more help at finite budgets even when the baseline is already optimizing the right eventual objective.

Our paper, *[Advancement or Acceleration? How Data Curation Improves LLM Training](https://arxiv.org/abs/2610.36983)*, develops this account using solvable teacher–student models of mixture and selection. Those models let us analyze finite-data learning curves and large-data endpoints separately. In the LLM experiments, we test whether the predicted behavior appears over the budgets we can afford; a persistent empirical gain is evidence consistent with advancement, rather than a measurement of an infinite-data limit.

The other two settings expose two kinds of gap. Holding out CommonCrawl creates an **exogenous gap**: the training sources and target differ. Repeatedly training on a finite corpus creates an **endogenous gap**: the learner fits the seen examples better than it generalizes to the target, even when those examples originally came from the same source distribution.

That distinction matters. Consuming more independent data and revisiting a fixed collection are different ways to extend training. The population no-gap argument concerns the former idealization; repetition-induced overfitting requires examining the finite corpus and training procedure as well.

In both cases, the useful intervention depends on what can be corrected. Reweighting can favor related sources or reduce repetition. It cannot create useful training material absent from every available source. A gap creates an opportunity only when the data and permitted operation can address it.

## What I check before searching for a better recipe

The disappearing win changed what I want to know before investing in data optimization.

First, I specify the target: the distribution and metric that define success. Then I examine the uncurated baseline and ask where its training exposure falls short.

One inexpensive clue is our **Δ-test**, which compares the baseline's loss on seen training data with its target loss, accounting for domain weights in mixture experiments. We measured it halfway through baseline training. It was small in the matched setting and larger in the two settings where curation retained gains. A small proxy model can help screen tasks before larger runs.

Δ is a diagnostic, not a verdict. A near-zero value can miss a real distributional difference; a nonzero value does not tell us whether an available policy can fix it. The formulas and measurement details are in the optional notes below.

Finally, I match the evaluation to the claim. For acceleration, I compare the total cost of reaching the same performance, including search and scoring overhead. For advancement, I extend the budget comparison and check whether the advantage survives as the baseline continues learning.

The 3.5-point improvement was useful evidence. It answered a question about a small training budget. Continuing the experiment revealed the next question: **was the new mixture helping the learner approach the same destination, or correcting something that would otherwise keep it from reaching a better one?**

That question also changes how I think about “good data.” Its value depends on the target and on what the rest of the training distribution already provides—the subject of [Part 2]({{ '/blog/2026/what-makes-data-good/' | relative_url }}).

## Technical notes (optional)

<details markdown="1">
<summary>Formal definitions and the population no-gap argument</summary>

Let $\mathcal E_\alpha(w)$ denote target loss under policy $w$ at data budget $\alpha$, with lower values better, and let $w_0$ be the baseline. Define

$$
\mathcal E_\infty(w)=\lim_{\alpha\to\infty}\mathcal E_\alpha(w).
$$

Advancement means $\mathcal E_\infty(w)<\mathcal E_\infty(w_0)$. For a performance level $\varepsilon$ attainable by both policies, define

$$
\alpha_w(\varepsilon)=\inf\{\alpha:\mathcal E_\alpha(w)\leq\varepsilon\}.
$$

Acceleration at that level means $\alpha_w(\varepsilon)<\alpha_{w_0}(\varepsilon)$. A policy can provide both effects.

The no-gap argument assumes a fixed model class, coincident population objectives for every model parameter vector $\theta$, and a global baseline minimizer:

$$
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0)
=\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta),
\qquad
\theta^*(w_0)\in\arg\min_\theta
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0).
$$

For any model $\theta^*(w)$ produced by another allocation in that class,

$$
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta^*(w_0))
\leq
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta^*(w)).
$$

This is a statement about the idealized population optimum. Reaching a stationary point in neural-network training does not establish its global-minimization premise. Equal losses at one checkpoint do not establish equality of the objective functions either.

</details>

<details markdown="1">
<summary>How we measure Δ</summary>

Let $\theta_{t_\Delta}(w_0)$ be the baseline checkpoint used for diagnosis. We measured Δ at 50% of the baseline training budget, without requiring convergence.

For mixture over $K$ domains, let $w_{0,k}$ denote baseline exposure and $\widehat w_k$ fixed target importance. Then

$$
\Delta_{\mathrm{mix}}
=\sum_{k=1}^K\left|
\widehat w_k\mathcal L_{\mathrm{target}}^k(\theta_{t_\Delta}(w_0))
-w_{0,k}\mathcal L_{\mathrm{train}}^k(\theta_{t_\Delta}(w_0))
\right|.
$$

For selection, the baseline retains the full pool, and

$$
\Delta_{\mathrm{sel}}
=\left|
\mathcal L_{\mathrm{target}}(\theta_{t_\Delta}(w_0))
-\mathcal L_{\mathrm{train}}(\theta_{t_\Delta}(w_0))
\right|.
$$

Training loss is evaluated on **seen training examples**. A held-out sample from the training source would miss the in-sample fitting effect we want to detect in the repetition-induced setting. Target loss estimates the designated target distribution; target data used to guide curation must be distinguished from independent final evaluation data.

In the mixture experiments, Δ was 0.07 in the matched setting, 0.64 with a restricted corpus, and 1.26 with the held-out CommonCrawl target. These values describe the studied settings; they do not define a universal threshold or a calibrated prediction of gain. Distinct distributions can give similar losses, and finite-sample variation can produce a nonzero diagnostic.

</details>
