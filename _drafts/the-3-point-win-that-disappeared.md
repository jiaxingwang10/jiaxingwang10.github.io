---
layout: post
title: "The 3.5-Point Win That Disappeared: Advancement vs. Acceleration"
date: 2026-10-13 12:00:00 +0800
description: "A data recipe can look much better at a fixed budget and still converge to the same model. How can we tell lasting advancement from temporary acceleration?"
tags: llm data-curation data-mixture data-selection scaling
categories: research
thumbnail: assets/img/blog/data-curation/advancement-acceleration.png
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Series draft v2. Add the Advancement paper/arXiv link after it becomes public. -->

We once had a data-mixture result that looked almost too clean.

At an intermediate checkpoint, the optimized mixture beat uniform sampling by **3.5 perplexity points**. If we had stopped the run there—as fixed-budget comparisons often do—we would have called it a convincing success.

Then we kept training.

The gap shrank. And shrank. At the largest budget, it was essentially gone.

Nothing was wrong with the experiment. The method really had helped. It had simply helped in a different way from what the early checkpoint suggested: it reached the same destination faster.

This is where I find a lot of discussion around data curation surprisingly slippery. We use the word *improvement* for two very different things:

> **Acceleration** changes how quickly we reach a level of performance.  
> **Advancement** changes the best level of performance we can eventually reach.

Both matter. Saving 30% of the tokens can be enormously valuable. But “we saved compute” and “we built a better model” are different claims, and a single checkpoint cannot tell them apart.

## The endpoint and the road to it

Let $\mathcal E_\alpha(w)$ be the target loss after training with data budget $\alpha$ under a curation policy $w$. The policy may represent domain-mixture weights, example-selection decisions, or token-level masks. Let $w_0$ be the uncurated baseline.

The endpoint is

$$
\mathcal E_\infty(w)=\lim_{\alpha\rightarrow\infty}\mathcal E_\alpha(w).
$$

A policy provides **advancement** if

$$
\mathcal E_\infty(w)<\mathcal E_\infty(w_0).
$$

For a target loss $\varepsilon$, let $\alpha_w(\varepsilon)$ be the smallest budget needed to reach it. The policy provides **acceleration** if

$$
\alpha_w(\varepsilon)<\alpha_{w_0}(\varepsilon).
$$

In pictures, acceleration bends the learning curve; advancement moves its endpoint.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*The result that started this line of thought. RegMix is ahead of Uniform by roughly 3.5 perplexity points early on, but the advantage vanishes with more training.*

This distinction changes how a result should be valued. If a method accelerates training, report saved tokens, FLOPs, and wall-clock time at matched performance. If it advances the model, show that the advantage survives increasing budgets. Otherwise, an early lead can quietly turn into an endpoint tie.

## Why would a data method change the endpoint at all?

Suppose I give you a gigantic jar containing red and blue marbles and ask you to reproduce the same red–blue ratio in another jar. A clever sampling strategy may help you estimate the ratio faster. But if ordinary sampling already targets the right distribution, no reweighting trick should produce a *more correct* answer in the limit.

Data curation has the same basic constraint.

Write curation as a bilevel problem:

$$
\min_w \; \mathcal L_{\mathrm{target}}\!\left(\theta^*(w)\right)
\quad\text{s.t.}\quad
\theta^*(w)\in\arg\min_\theta \mathcal L_{\mathrm{train}}(\theta;w).
$$

The inner problem trains the model. The outer problem changes the data policy so that the trained model performs better on a target.

Now imagine that the baseline training objective is already the target objective. With enough data and successful optimization, the baseline model minimizes exactly what we evaluate. Rearranging the same support can make learning easier, but it has no mismatch to repair at the endpoint.

### The one-line obstruction

This intuition can be made precise without using anything specific to language models—or even to our perceptron analysis.

Let $w_0$ denote the uncurated baseline and suppose there is no train–target gap at the population level:

$$
\mathcal L_{\mathrm{train}}(\theta;w_0)
=
\mathcal L_{\mathrm{target}}(\theta)
\qquad \text{for every }\theta.
$$

Because the inner optimization converges, the baseline solution

$$
\theta_0^*\in\arg\min_\theta
\mathcal L_{\mathrm{train}}(\theta;w_0)
$$

also globally minimizes the target objective. For any alternative curation policy $w$, its trained solution $\theta^*(w)$ is still just another model in the same hypothesis class. Therefore

$$
\mathcal L_{\mathrm{target}}(\theta_0^*)
=
\min_\theta \mathcal L_{\mathrm{target}}(\theta)
\leq
\mathcal L_{\mathrm{target}}\!\left(\theta^*(w)\right).
$$

That is the whole obstruction: **when the baseline already optimizes the target population objective, no reallocation can produce a strictly better target endpoint.** Another policy may tie the baseline, reach the endpoint faster, or optimize more reliably at finite scale. But it cannot lower the idealized endpoint without breaking one of the premises—most importantly, without revealing or creating a train–target gap.

The assumptions matter. This is an endpoint statement under population objectives, a fixed model class, and successful inner optimization. Finite data, optimization error, regularization, or a different feasible support can all change the story. Those are not counterexamples; they tell us which premise supplied the headroom.

This leads to the central claim of our recent work:

> **Lasting advancement requires a correctable train–target gap.**

The adjective *correctable* matters. A mismatch alone is not enough. The candidate data must contain a direction that helps, and the allowed operation—mixture, selection, or something else—must be capable of moving the effective training distribution in that direction.

So the logic is asymmetric:

- No gap: curation may accelerate, but it should not advance the endpoint.
- A gap: advancement becomes possible, but is not guaranteed.

The figure below is the map of the argument. Read it from left to right: setup, acceleration, then advancement. Read it by row: mixture on top, selection below. Solid curves are the analytic prediction; circles are finite-dimensional simulations.

{% include figure.liquid path="assets/img/blog/data-curation/advancement-theory-map.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*The same structure appears for two different curation operators. With no gap, changing the allocation can improve finite-budget learning but not the endpoint. Once a correctable gap exists, mixture matching or selective removal can produce lasting advancement.*

## Mixture and selection tell the same story

The mechanism appears in two seemingly different problems.

### Mixture: which domains receive the next token?

In data-mixture optimization, $w$ allocates a budget across domains. At finite budgets, harder or slower-learning domains may deserve more samples. That can create acceleration even when the baseline mixture matches the target.

But the endpoint is less forgiving. If the baseline already represents the target distribution, another mixture has nothing systematic to correct. If the target emphasizes a different composition—or limited data causes some domains to be over-repeated—reallocation can repair that mismatch and change the endpoint.

### Selection: which examples survive?

Selection has an extra tension. Filtering can remove misleading examples, but it also throws away training signal.

With no mismatch, retaining everything is endpoint-optimal under our assumptions; sufficiently aggressive filtering eventually hurts. With a correctable mismatch, an interior keep ratio can become optimal: the gain from better alignment exceeds the loss from using fewer samples.

The interesting question is therefore not “Does selection work?” or “Does mixture optimization work?” It is:

> **What mismatch is this operator able to correct?**

## A one-run test before an expensive search

The distribution-level statement is neat, but real data distributions are not sitting in a folder waiting to be compared. We therefore wanted a diagnostic that could be computed from an ordinary baseline run.

For selection, define

$$
\Delta_{\mathrm{sel}}
=
\left|
\mathcal L_{\mathrm{target}}(\theta(w_0))
-
\mathcal L_{\mathrm{train}}(\theta(w_0))
\right|.
$$

For a $K$-domain mixture, compare target and training contributions domain by domain:

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

We call this the **$\Delta$-test**. It is deliberately modest: not a universal distance between distributions, and certainly not a proof that an algorithm will work. It asks a cheaper question first—does the baseline expose enough target mismatch to justify an expensive data-optimization project?

A near-zero $\Delta$ means weak observed headroom for advancement. A noticeable $\Delta$ means there may be something to correct. We still need to check that the candidate pool and operator can reach it.

## The same algorithms, three different worlds

We tested this idea by changing the curation task while holding the algorithm family largely fixed.

For data mixture, we trained GPT-style models on seven SlimPajama domains and compared Uniform with Skill-It, RegMix, Aioli, DoGE, and TANDEM.

In the matched setting, $\Delta=0.07$. The optimized methods finished between 19.66 and 20.17 perplexity, statistically close to Uniform at 19.71. Some methods looked better early. None meaningfully moved the endpoint.

Then we introduced two gaps:

- an **endogenous gap** from limited domain data and repeated sampling, with $\Delta=0.64$;
- an **exogenous gap** from evaluating on held-out CommonCrawl, with $\Delta=1.26$.

The algorithms did not suddenly become smarter. The task became correctable. In both gap settings, optimized mixtures kept substantial gains at the largest budget. Under the exogenous gap, target perplexity fell from 49.19 for Uniform to roughly 40–41.

{% include figure.liquid path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*In the matched regime, mixture optimization mostly changes the path. Once a correctable gap appears, the same family of methods can change the endpoint.*

Selection gave us an even cleaner contrast. We kept the model, candidate pool, budget, and selectors fixed, and changed only the target.

The pool contained 5B OpenWebMath tokens, and every selector retained 60%. When the target came from the same distribution, $\Delta=0.016$; full-pool training was hard to beat. When the target shifted to MetaMath–Mammoth, $\Delta=0.549$; target-aware methods now had something concrete to correct. BLADE raised average benchmark accuracy from 17.21 to 31.31 while using only 60% of the candidate tokens.

{% include figure.liquid path="assets/img/blog/data-curation/selection-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Only the target changed. The usefulness of selection changed with it.*

This paired result is more revealing than a leaderboard win. Access to a target set does not automatically make target-aware selection useful. It becomes useful when the target reveals a mismatch that the candidate pool can repair.

## A different order of operations

The usual workflow is:

1. invent a scoring rule;
2. select or reweight data;
3. train a model;
4. hope the target metric improves.

I think the order should be reversed:

1. **Define the target.** What distribution and metrics represent the capability we actually care about?
2. **Train the boring baseline.** Let it run long enough to see whether early gaps are closing.
3. **Diagnose the headroom.** Measure $\Delta$, ideally at several checkpoints and first with a smaller proxy model.
4. **Check correctability.** Does the pool contain the missing pattern, and can the available operator expose it?
5. **Only then optimize the data.**

This is not an argument against data curation. It is an argument for being precise about what data curation accomplished.

Sometimes a method buys compute. Sometimes it changes the model we can obtain. The most misleading case is when we mistake the first for the second because we stopped the learning curve too early.

And this leads to a deeper question. If the value of curation depends on a target and a correctable mismatch, can a sample be called “high quality” before either is specified?

That is the subject of the [next essay]({{ '/blog/2026/what-makes-data-good/' | relative_url }}).
