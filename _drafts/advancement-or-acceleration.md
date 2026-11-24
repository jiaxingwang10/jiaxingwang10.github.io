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

Let \(\mathcal E_\alpha(w)\) denote target loss after training with data budget \(\alpha\) under a curation policy \(w\). Depending on the problem, \(w\) may represent domain mixture weights or per-example selection decisions. The superscript or subscript $\mathrm{base}$ denotes the uncurated baseline: uniform allocation for mixture, or full retention for selection.

The limiting target loss is

$$
\mathcal E_\infty(w)=\lim_{\alpha\rightarrow\infty}\mathcal E_\alpha(w).
$$

A policy provides **advancement** when

$$
\mathcal E_\infty(w)<\mathcal E_\infty^{\mathrm{base}}.
$$

For a target loss level \(\varepsilon\), define \(\alpha_w(\varepsilon)\) as the smallest budget needed by policy \(w\) to reach it. The same policy provides **acceleration** when

$$
\alpha_w(\varepsilon)<\alpha_{\mathrm{base}}(\varepsilon).
$$

The distinction is easiest to see through learning curves. Acceleration changes the path: one curve initially falls faster, but the curves eventually meet. Advancement changes the endpoint: the advantage remains as the training budget grows.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*A real mixture experiment with a negligible gap. RegMix is ahead of Uniform by roughly 3.5 perplexity points early in training, but the advantage vanishes with a larger budget. This is acceleration, not advancement.*

This is not merely a terminological distinction. If a method only accelerates training, its value should be measured by saved FLOPs, wall-clock time, or tokens. If it advances the model, then it has changed the attainable capability under the task. A single fixed-budget comparison cannot tell us which one happened.

## When can curation change the endpoint?

Our central finding is that advancement requires a **correctable train–target gap**.

Let \(\mathcal P_{\mathrm{train}}\) be the effective distribution induced by the uncurated training policy, and let \(\mathcal P_{\mathrm{target}}\) describe what we ultimately care about. A gap exists when these distributions—or, more precisely, their induced objectives—do not coincide. It is *correctable* when an available curation operation can reduce the mismatch.

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

For **data mixture**, the training budget is divided across domains according to \(w\), while the target uses a fixed composition \(\widehat w\). At finite budgets, harder domains may deserve more samples because they learn more slowly. That can produce acceleration even when \(w=\widehat w\). But at the large-budget endpoint, matching the training mixture to the target is optimal. When the baseline already matches the target, no alternative mixture advances it. When \(w\neq\widehat w\), reallocation can close the gap and improve the endpoint.

For **data selection**, there is an additional trade-off. Filtering removes training signal, but it may also remove examples that are inconsistent with the target. With no mismatch, full retention remains endpoint-optimal; aggressive pruning eventually hurts. With a correctable mismatch, an interior keep ratio can be optimal: the gain from improved alignment outweighs the loss of sample size.

The lesson is not that mixture or selection is intrinsically effective. Their value depends on the task in which they operate.

## Diagnosing headroom before optimizing data

The population condition is clean but cannot usually be checked directly on real corpora. We therefore use the **$\Delta$-diagnostic** from an ordinary uncurated baseline run. Let $\theta$ denote a checkpoint from that run; training need not have converged.

First, match each of the $J$ target components $j$ to a group $S_j$ of training domains. Every training domain belongs to exactly one group. With target weights $\widehat w_j$ and baseline training proportions $w_k$, define the signed discrepancy

$$
d_j(\theta)
=
\widehat w_j\mathcal L_{\mathrm{target}}^j(\theta)
-
\sum_{k\in S_j}w_k\mathcal L_{\mathrm{train}}^k(\theta).
$$

**The grouping depends on the task.** When training and target domains correspond one to one (*paired*, $J=K$), use $S_j=\{j\}$. For a held-out target domain with no counterpart in the candidate pool (*unpaired*), use one target component and group all training domains together: $J=1$, $S_1=\{1,\ldots,K\}$. This compares the held-out target loss with the baseline's weighted training loss.

**Correct for intrinsic data difficulty.** Raw loss differences also reflect the intrinsic entropy of the data, especially when the target and training sources are unpaired. We subtract the same signed discrepancy measured by a reference model $\theta_{\mathrm{ref}}$ on the **same documents**, then take absolute values:

$$
\Delta
=
\sum_{j=1}^{J}
\left|d_j(\theta)-d_j(\theta_{\mathrm{ref}})\right|.
$$

The reference should capture intrinsic data difficulty and must not have been trained on the baseline's training data.

Selection is the single-pool case, $K=J=1$, with full retention as the baseline:

$$
\Delta
=
\left|
\bigl[\mathcal L_{\mathrm{target}}(\theta)-\mathcal L_{\mathrm{train}}(\theta)\bigr]
-
\bigl[\mathcal L_{\mathrm{target}}(\theta_{\mathrm{ref}})-\mathcal L_{\mathrm{train}}(\theta_{\mathrm{ref}})\bigr]
\right|.
$$

Here, training loss is measured on examples already seen by the baseline, so the diagnostic can capture gaps caused by repeated exposure.

We calibrate $\Delta$ against a no-gap control within each experiment family, using held-out target data from the training distribution. An appreciably nonzero value means $\Delta>\mu_{\mathrm{null}}+3\sigma_{\mathrm{null}}$, where the null mean and standard deviation come from independently trained baselines. Its scale depends on the model, tokenizer, and data, so there is no universal raw-value threshold.

The diagnostic signals **potential** for advancement. A nonzero value does not guarantee that the available data and policy can correct the mismatch. Conversely, scalar losses can miss distributional differences, so a near-zero value is not proof that no gap exists.

## What happens in real LLM training?

We tested the prediction in controlled mixture and selection experiments. The goal was not merely to compare algorithms, but to vary whether a correctable gap existed while keeping the rest of the setup fixed.

### Mixture: the same algorithms behave differently in different tasks

We trained GPT-style language models on seven SlimPajama domains and compared Uniform with Skill-It, RegMix, Aioli, DoGE, and TANDEM.

In the matched setting, \(\Delta=0.076\). Final perplexities for all optimized mixtures fell between 19.66 and 20.17, statistically close to Uniform at 19.71. Several methods looked better earlier, but their gains disappeared.

We then introduced two kinds of gap:

- An **endogenous gap** caused by limited domain data and repeated sampling, with \(\Delta=0.649\).
- An **exogenous gap** created by evaluating on a held-out CommonCrawl target, with \(\Delta=0.468\). Although CommonCrawl was absent from training, the related C4 domain provided a feasible correction direction.

In both cases, optimized mixtures retained substantial gains at the largest tested budget. Under the exogenous gap, target perplexity dropped from 49.19 for Uniform to roughly 40–41 for the optimized methods.

{% include figure.liquid path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Mixture optimization has almost no final gain in the matched regime. The same family of methods becomes effective when limited data or distribution shift creates a correctable gap.*

The diagnosis also transferred across scale. Moving from 160M to 1.1B parameters preserved the qualitative separation: negligible gains in the no-gap setting, and substantial gains in both gap settings. This supports using a smaller baseline model to screen for headroom before launching a full-scale search.

### Selection: target awareness matters only when there is something to correct

For selection, we held the model, candidate pool, training budget, and selection methods fixed. We changed only the target distribution.

The candidate pool contained 5B OpenWebMath tokens. Every selector retained 60%. In the no-gap setting, the target was drawn from the same distribution, producing \(\Delta=0.016\). Full-pool training remained difficult to beat, and most pruning strategies lost accuracy.

In the shifted setting, the target was MetaMath–Mammoth and \(\Delta=0.158\). Target-agnostic heuristics still failed to help, while target-aware RHO-1 and BLADE produced large gains. BLADE increased average benchmark accuracy from 17.21 for full-pool continual training to 31.31 using only 60% of the candidate tokens.

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
