---
layout: post
published: false
title: "Does Data Curation Really Improve Models? Advancement vs. Acceleration"
date: 2026-10-13
description: "A fixed-budget gain does not necessarily mean a better model. We distinguish lasting advancement from transient acceleration and ask when data curation can change the attainable endpoint."
tags: llm data-curation data-mixture data-selection scaling
categories: research
thumbnail: assets/img/blog/data-curation/mixture-gap-results.png
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Draft: add the Advancement paper/arXiv link after it becomes public. -->

Data curation is usually evaluated in a simple way: fix a training budget, apply a selection or mixture method, and compare the final number against a baseline. If the curated run is better, we say that the data method improved the model.

But this conclusion can be wrong.

In one of our experiments, an optimized data mixture reduced perplexity by **3.5 points** at an intermediate checkpoint. It looked like a decisive win. As training continued, however, the advantage steadily disappeared. At the largest budget, the optimized mixture and the uniform baseline reached essentially the same result.

The method had not improved what the model could ultimately learn. It had only helped the model get there faster.

This observation motivates a distinction that I think is missing from much of the data-curation literature:

> **Advancement** changes the best target performance a model can attain.  
> **Acceleration** reaches the same target performance with less data or compute.

Both are useful. But they are not the same scientific claim, and they imply very different decisions in practice.

## The endpoint and the path

Let \(\mathcal E_\alpha(w)\) denote target loss after training with data budget \(\alpha\) under a curation policy \(w\). Depending on the problem, \(w\) may represent domain mixture weights or per-example selection decisions. Let \(w_0\) be the uncurated baseline: uniform allocation for mixture, or full retention for selection.

The limiting target loss is

$$
\mathcal E_\infty(w)=\lim_{\alpha\rightarrow\infty}\mathcal E_\alpha(w).
$$

A policy provides **advancement** when

$$
\mathcal E_\infty(w)<\mathcal E_\infty(w_0).
$$

For a target loss level \(\varepsilon\), define \(\alpha_w(\varepsilon)\) as the smallest budget needed by policy \(w\) to reach it. The same policy provides **acceleration** when

$$
\alpha_w(\varepsilon)<\alpha_{w_0}(\varepsilon).
$$

The distinction is easiest to see through learning curves. Acceleration changes the path: one curve initially falls faster, but the curves eventually meet. Advancement changes the endpoint: the advantage remains as the training budget grows.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*A real mixture experiment with a negligible gap. RegMix is ahead of Uniform by roughly 3.5 perplexity points early in training, but the advantage vanishes with a larger budget. This is acceleration, not advancement.*

This is not merely a terminological distinction. If a method only accelerates training, its value should be measured by saved FLOPs, wall-clock time, or tokens. If it advances the model, then it has changed the attainable capability under the task. A single fixed-budget comparison cannot tell us which one happened.

## When can curation change the endpoint?

Our central finding is that advancement requires a **correctable train–target gap**.

Let \(\mathcal P_{\mathrm{train}}(w_0)\) be the effective distribution induced by the uncurated training policy, and let \(\mathcal P_{\mathrm{target}}\) describe what we ultimately care about. A gap exists when these distributions—or, more precisely, their induced objectives—do not coincide. It is *correctable* when an available curation operation can reduce the mismatch.

Why is a gap necessary? Consider the bilevel view of curation:

$$
\min_w \; \mathcal L_{\mathrm{target}}\!\left(\theta^*(w)\right)
\quad\text{s.t.}\quad
\theta^*(w)\in\arg\min_\theta \mathcal L_{\mathrm{train}}(\theta;w).
$$

If the population training objective under the baseline is already identical to the target objective, the baseline-trained model globally minimizes the very quantity used for evaluation. Reallocating the same support cannot produce a lower target endpoint. It may improve finite-budget optimization, but it has nothing to correct asymptotically.

This gives a useful negative result:

> **Without a train–target gap, curation can accelerate learning, but it cannot advance the endpoint.**

The converse is deliberately weaker. A nonzero gap creates headroom; it does not guarantee that a particular method will exploit it. The candidate pool must contain useful directions, and the chosen operator—mixture or selection—must be expressive enough to move the training distribution toward the target.

## The same mechanism appears in mixture and selection

We first studied the distinction in an analytically tractable teacher–student model.

For **data mixture**, the training budget is divided across domains according to \(w\), while the target uses a fixed composition \(\widehat w\). At finite budgets, harder domains may deserve more samples because they learn more slowly. That can produce acceleration even when \(w_0=\widehat w\). But at the large-budget endpoint, matching the training mixture to the target is optimal. When the baseline already matches the target, no alternative mixture advances it. When \(w_0\neq\widehat w\), reallocation can close the gap and improve the endpoint.

For **data selection**, there is an additional trade-off. Filtering removes training signal, but it may also remove examples that are inconsistent with the target. With no mismatch, full retention remains endpoint-optimal; aggressive pruning eventually hurts. With a correctable mismatch, an interior keep ratio can be optimal: the gain from improved alignment outweighs the loss of sample size.

The lesson is not that mixture or selection is intrinsically effective. Their value depends on the task in which they operate.

## Diagnosing headroom before optimizing data

The distribution-level condition above is conceptually clean but difficult to measure directly. We therefore introduce a lightweight empirical diagnostic, the **\(\Delta\)-test**, computed from one uncurated baseline run.

For data selection, it is simply the discrepancy between target loss and training loss:

$$
\Delta_{\mathrm{sel}}
=
\left|
\mathcal L_{\mathrm{target}}(\theta(w_0))
-
\mathcal L_{\mathrm{train}}(\theta(w_0))
\right|.
$$

For a \(K\)-domain mixture, the diagnostic compares the domain-wise target and training contributions:

$$
\Delta_{\mathrm{mix}}
=
\sum_{k=1}^{K}
\left|
\widehat w_k\mathcal L_{\mathrm{target}}^k(\theta(w_0))
-
w_k\mathcal L_{\mathrm{train}}^k(\theta(w_0))
\right|.
$$

This is an operational signal, not a universal statistical distance. An appreciably nonzero \(\Delta\) says that the baseline exposes a mismatch worth investigating. A near-zero value says that increasingly sophisticated curation is unlikely to change the endpoint, although it may still save compute.

There are two important qualifications:

1. **A nonzero \(\Delta\) is necessary evidence, not a sufficient guarantee.** The feasible data operation must still be able to correct the gap.
2. **A scalar loss cannot reveal every possible distributional mismatch.** A near-zero diagnostic should be interpreted as weak observed headroom, not as a proof that the distributions are identical.

Despite these limitations, the test changes the order of operations: diagnose first, optimize second.

## What happens in real LLM training?

We tested the prediction in controlled mixture and selection experiments. The goal was not merely to compare algorithms, but to vary whether a correctable gap existed while keeping the rest of the setup fixed.

### Mixture: the same algorithms behave differently in different tasks

We trained GPT-style language models on seven SlimPajama domains and compared Uniform with Skill-It, RegMix, Aioli, DoGE, and TANDEM.

In the matched setting, \(\Delta=0.07\). Final perplexities for all optimized mixtures fell between 19.66 and 20.17, statistically close to Uniform at 19.71. Several methods looked better earlier, but their gains disappeared.

We then introduced two kinds of gap:

- An **endogenous gap** caused by limited domain data and repeated sampling, with \(\Delta=0.64\).
- An **exogenous gap** created by evaluating on a held-out CommonCrawl target, with \(\Delta=1.26\). Although CommonCrawl was absent from training, the related C4 domain provided a feasible correction direction.

In both cases, optimized mixtures retained substantial gains at the largest tested budget. Under the exogenous gap, target perplexity dropped from 49.19 for Uniform to roughly 40–41 for the optimized methods.

{% include figure.liquid path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Mixture optimization has almost no final gain in the matched regime. The same family of methods becomes effective when limited data or distribution shift creates a correctable gap.*

The diagnosis also transferred across scale. Moving from 160M to 1.1B parameters preserved the qualitative separation: negligible gains in the no-gap setting, and substantial gains in both gap settings. This supports using a smaller baseline model to screen for headroom before launching a full-scale search.

### Selection: target awareness matters only when there is something to correct

For selection, we held the model, candidate pool, training budget, and selection methods fixed. We changed only the target distribution.

The candidate pool contained 5B OpenWebMath tokens. Every selector retained 60%. In the no-gap setting, the target was drawn from the same distribution, producing \(\Delta=0.016\). Full-pool training remained difficult to beat, and most pruning strategies lost accuracy.

In the shifted setting, the target was MetaMath–Mammoth and \(\Delta=0.549\). Target-agnostic heuristics still failed to help, while target-aware RHO-1 and BLADE produced large gains. BLADE increased average benchmark accuracy from 17.21 for full-pool continual training to 31.31 using only 60% of the candidate tokens.

{% include figure.liquid path="assets/img/blog/data-curation/selection-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Changing only the target changes the value of selection. Target-aware methods have little endpoint headroom in the matched setting and large gains after a target shift.*

This paired comparison is important. Access to a target set does not automatically make a selection method useful. Target awareness becomes valuable when the target reveals a correctable mismatch.

## A practical decision rule

Before building another data scorer or running an expensive mixture search, I would ask four questions:

1. **What is the target?** Define a held-out distribution and metrics that represent the capability you actually want.
2. **What does the uncurated baseline attain?** Train it long enough to see whether an early gain is shrinking.
3. **Is there observable headroom?** Measure \(\Delta\) from the baseline, preferably at several checkpoints and with a smaller proxy model first.
4. **Can the available operator correct the mismatch?** Mixture and selection can only redistribute mass over existing support. If the required behavior is absent, the right action may be data synthesis rather than more elaborate ranking.

This framing also clarifies what should be reported. A curation paper claiming acceleration should report tokens, FLOPs, or wall-clock saved at matched performance. A paper claiming advancement should show that its gain persists across increasing budgets and should explain what mismatch it corrects.

## The broader implication

The main lesson is not that data curation is ineffective. It is that its success is conditional.

Advancement is not solely a property of the algorithm. It is a property of the **curation task**: the target, the baseline training distribution, the candidate support, and the allowed operation. This helps explain why sophisticated methods sometimes collapse to uniform or random baselines, while the same ideas work extremely well under distribution shift or limited data.

It also elevates target construction from an evaluation detail to a central design decision. Candidate training pools may contain trillions of tokens, but target sets are compact and controllable. They define which gap can be observed, which direction data optimization follows, and what “better data” ultimately means.

The next essay takes this argument one step further: if data value is target-relative, then data curation is best understood not as assigning an absolute quality score, but as **designing a training distribution**.
