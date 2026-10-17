---
layout: post
title: "The 3.5-Point Win That Disappeared: Advancement vs. Acceleration"
date: 2026-10-13 12:00:00 +0800
description: "A data recipe can look much better at a fixed budget and still converge to the same model. How can we tell lasting advancement from temporary acceleration?"
tags: llm data-curation data-mixture data-selection scaling
categories: research
thumbnail: assets/img/blog/data-curation/advancement-acceleration.png
series: designing-what-models-learn
series_title: "Designing What Models Learn"
series_order: 1
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Series draft v3. Add the Advancement paper/arXiv link after it becomes public. -->

> **Designing What Models Learn · Part 1 of 3**<br>
> *Essays on targets, training distributions, and data operators*<br>
> **Advancement vs. Acceleration** · [What Makes Data Good?]({{ '/blog/2026/what-makes-data-good/' | relative_url }}) · [From Target to Operator]({{ '/blog/2026/from-target-to-operator/' | relative_url }})

We had been trying to improve a language model's capability by optimizing its training-data mixture. The hope was simple: if we allocated the token budget more intelligently across domains, perhaps the resulting model would become genuinely better.

For one experiment, we followed the setup from [Aioli](https://arxiv.org/abs/2411.05735): train a 160M-parameter GPT-style model on seven [SlimPajama](https://www.cerebras.net/blog/slimpajama-a-627b-token-cleaned-and-deduplicated-version-of-redpajama) domains—ArXiv, Books, C4, CommonCrawl, GitHub, StackExchange, and Wikipedia. A mixture is evaluated by the model's held-out loss on all seven domains, summarized as an averaged perplexity: the exponential of the mean domain loss.

We tried [RegMix](https://arxiv.org/abs/2407.01492), which trains small proxy models on different mixtures, fits a regression model to their outcomes, and uses it to propose a promising mixture for a larger run. We then trained a fresh model under that RegMix-optimized mixture.

At 2,000 training steps, this new model beat the one trained with a uniform mixture by **3.5 averaged-perplexity points**. It looked like exactly the result we had hoped for. If we had stopped there—as fixed-budget comparisons often do—we would have called it a convincing success.

Then we kept the same run going, consuming more training data.

The gap shrank. And shrank. By the end, it was gone.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*RegMix is 3.5 perplexity points better at 2,000 steps. At the final checkpoint it is slightly worse than Uniform: the important observation is not that the two runs become identical, but that RegMix no longer retains an advantage.*

That disappearing win left me with a question that eventually became the focus of this work:

> **When data curation improves a language model, does it change what the model can ultimately learn, or only how quickly it gets there?**

The result was real, but its meaning was narrower than the early checkpoint suggested. RegMix reached a given performance sooner without producing a better attainable model.

This is where I find a lot of discussion around data curation surprisingly slippery. We use the word *improvement* for two different things:

> **Acceleration** changes how quickly we reach a level of performance.  
> **Advancement** changes the best level of performance we can eventually reach.

I do not think these two improvements carry the same scientific weight. A large and persistent reduction in compute can be genuinely valuable. But a small advantage at an early checkpoint often says only that the baseline has not caught up. In large-scale training, saved tokens are usually reinvested rather than left unused, so the more consequential question is whether data curation changes the best model we can obtain after the available budget has been spent.

My own priority is therefore clear:

> **advancement $>$ acceleration.**

Acceleration is compelling when it behaves like a real compute multiplier: the same capability can be reached with substantially fewer tokens across meaningful scales. But if the advantage disappears as the baseline consumes more data, then the claimed acceleration was not robust enough to matter. It did not produce a better attainable model.

## The endpoint and the road to it

Let $\mathcal E_\alpha(w)$ denote target performance after training with data budget $\alpha$ under curation policy $w$. The policy may represent domain-mixture weights, example-selection decisions, or token-level masks. To keep the inequalities concrete, write $\mathcal E$ in a lower-is-better orientation—loss, error, or negative accuracy. For an accuracy metric, simply reverse the inequalities.

The endpoint is

$$
\mathcal E_\infty(w)=\lim_{\alpha\rightarrow\infty}\mathcal E_\alpha(w).
$$

A policy provides **advancement** if

$$
\mathcal E_\infty(w)<\mathcal E_\infty(w_0),
$$

where $w_0$ indicates the uncurated baseline.

For a target performance level $\varepsilon$, let $\alpha_w(\varepsilon)$ be the smallest budget needed to reach it. The policy provides **acceleration** if

$$
\alpha_w(\varepsilon)<\alpha_{w_0}(\varepsilon).
$$

Acceleration changes the road: the same performance is reached with less data. Advancement changes the destination: the endpoint itself improves. A single checkpoint cannot distinguish the two.

## Why would a data method change the endpoint at all?

Suppose I give you a gigantic jar containing red and blue marbles and ask you to reproduce the same red–blue ratio in another jar. A clever sampling strategy may help you estimate the ratio faster. But if ordinary sampling already targets the right distribution, no reweighting trick should produce a *more correct* answer in the limit.

Data curation has the same basic constraint. Write it as a bilevel problem:

$$
\min_w \; \mathcal L_{\mathrm{target}}\!\left(\theta^*(w)\right)
\quad\text{s.t.}\quad
\theta^*(w)\in\arg\min_\theta \mathcal L_{\mathrm{train}}(\theta;w).
$$

The inner problem trains the model. The outer problem changes the data policy so that the trained model performs better on a target.

A different mixture or subset can still make finite-data learning more efficient—for example, by showing the learner more informative examples earlier. That is acceleration. But if the uncurated **population** training objective already equals the target objective, then at the endpoint there is no distributional mismatch left for reallocation to correct.

### The one-line obstruction

Let $w_0$ denote the uncurated baseline and suppose, crucially, that there is no train–target gap **at the population level**:

$$
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0)
=
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta)
\qquad \text{for every }\theta.
$$

This is much stronger than observing equal training and target losses at one checkpoint. It says that the two expected objectives are the same function of the model. The population solution is the large-data endpoint represented by $\alpha\to\infty$.

Because the inner optimization converges, the baseline solution

$$
\theta^*(w_0)\in\arg\min_\theta
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0)
$$

also globally minimizes the target objective. For any alternative curation policy $w$, its trained solution $\theta^*(w)$ is still another model in the same hypothesis class. Therefore

$$
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}\!\left(\theta^*(w_0)\right)
=
\min_\theta \mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta)
\leq
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}\!\left(\theta^*(w)\right).
$$

That is the obstruction: **when the uncurated baseline already optimizes the target population objective, no reallocation can produce a strictly better target endpoint.** Another policy may reach it with fewer samples, but it cannot lower the endpoint.

The statement concerns a population endpoint within a fixed model class and a converged inner optimization. At finite budgets, optimization and sampling effects remain—and those are exactly where acceleration lives.

These results are developed in our forthcoming paper, **Advancement or Acceleration? How Data Curation Improves LLM Training**.

<!-- Add the public arXiv link to the paper title above once available. -->

> **Lasting advancement requires a correctable train–target gap.**

The adjective *correctable* matters. A mismatch alone is not enough. The candidate data must contain useful directions, and the permitted operation must be able to move the effective training distribution toward the target.

- No population gap: curation may accelerate, but it cannot improve the idealized endpoint under these assumptions.
- A correctable gap: advancement becomes possible, but is not guaranteed.

## A solvable map of mixture and selection

A small notation warning: the next figure is the densest part of this essay. Four symbols are enough to read it:

- $\alpha=P/N$ is the sample-to-parameter ratio, our normalized data budget.
- $w_1$ is the fraction of training data assigned to domain 1; $\widehat w_1$ is domain 1's fixed weight in the target distribution.
- $f$ is the fraction of candidate examples retained by selection.
- $\phi$ is the angle between the training teacher and target teacher; larger $\phi$ means a larger train–target gap.

We constructed teacher–student perceptron models for both curation operators. A teacher generates labels, a student is trained on the curated data, and the replica method gives an analytic relationship between the curation policy, the budget, and the trained student's target performance—precisely $\mathcal E_\alpha(w)$. This lets us study the large-data endpoint rather than extrapolate it from a short learning curve. Solid lines below are replica predictions; circles are simulations with $N=200$. The full derivations belong in the paper, but the experimental logic is visible in the six panels.

For **mixture** (top row), we use two domains. The policy $(w_1,1-w_1)$ controls their training proportions, while $(\widehat w_1,1-\widehat w_1)$ defines how the target evaluates them. Domain 2 is deliberately made harder without changing its labeling rule. Panel (b) fixes the matched target $\widehat w_1=0.5$: tilting toward the harder domain can help at small $\alpha$, but the curves meet as $\alpha$ grows. Panel (c) varies the target composition and shows that the large-budget optimum occurs at $w_1=\widehat w_1$. Reweighting changes the endpoint only when it corrects a mismatch between the baseline mixture and the target mixture.

For **selection** (bottom row), a training teacher $B$ labels the candidate pool and a target teacher $B_{\mathrm{tar}}$ defines evaluation. Their angle $\phi$ controls the gap. The policy retains a fraction $f$, favoring informative examples near the target decision boundary and filtering examples whose training labels conflict with the target teacher. At $\phi=0$, moderate selection can preserve accuracy with fewer examples, but it never beats full retention $f=1$. When $\phi>0$, an interior keep fraction $f^*<1$ can outperform full retention because the gain in target alignment outweighs the loss of training signal.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-theory-map.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Read left to right as setup, acceleration, and advancement; read top to bottom as mixture and selection. The same policy can have a different endpoint effect depending on whether a correctable gap exists.*

This also clarifies what the word *operator* is doing here. Mixture can only redistribute probability mass among existing domains. Selection can only retain or remove examples already in the candidate pool. If the target requires a mode absent from that pool, neither operation can create it. The practical question is therefore:

> **Given the observed gap, can the operation available to us actually move the training distribution in the required direction?**

## A one-run diagnostic before an expensive search

The population condition is clean but cannot usually be checked directly on real corpora. We therefore use the **$\Delta$-test** as an observable diagnostic from an ordinary uncurated baseline run.

For a $K$-domain mixture,

$$
\Delta
=
\sum_{k=1}^{K}
\left|
\widehat w_k\mathcal L_{\mathrm{target}}^k\!\left(\theta^*(w_0)\right)
-
w_{0,k}\mathcal L_{\mathrm{train}}^k\!\left(\theta^*(w_0)\right)
\right|.
$$

Here $\widehat w_k$ is domain $k$'s importance in the target distribution, while $w_{0,k}$ is its exposure under the uncurated training mixture. The domain-wise absolute values prevent one over-served domain from cancelling another under-served one.

For selection, there is a single candidate pool, so the diagnostic reduces to

$$
\Delta
=
\left|
\mathcal L_{\mathrm{target}}\!\left(\theta^*(w_0)\right)
-
\mathcal L_{\mathrm{train}}\!\left(\theta^*(w_0)\right)
\right|,
$$

where $w_0$ means full retention.

$\Delta$ is not a distance between distributions and it is not a sufficient condition for advancement. An appreciably nonzero value exposes a mismatch that a suitable policy may be able to correct. A near-zero value is more limited: it only says that this scalar loss comparison did not reveal a gap. Two different distributions can happen to produce the same loss, so $\Delta\approx0$ does **not** prove population matching. The population gap is the necessary condition; the $\Delta$-test is a cheap, potentially false-negative diagnostic for it.

## Controlled interventions on the gap

To test the account, we did more than compare algorithms on one benchmark. We applied largely the same mixture optimizers and training protocol to three deliberately different task regimes: one approximately matched regime and two regimes with correctable gaps. This lets us ask whether the same method family behaves differently when the structure of the task changes.

For mixture, we compared Uniform with [Skill-It](https://arxiv.org/abs/2307.14430), [RegMix](https://arxiv.org/abs/2407.01492), [Aioli](https://arxiv.org/abs/2411.05735), [DoGE](https://arxiv.org/abs/2310.15393), and [TANDEM](https://openreview.net/pdf?id=szBFUtBzWP):

- **Matched regime ($\Delta=0.07$).** Final averaged perplexities ranged from 19.66 to 20.17, statistically close to Uniform at 19.71. Early leads did not become a reliable endpoint gain.
- **Endogenous gap ($\Delta=0.64$).** Restricting the pool forced repeated sampling of data-scarce domains; optimized mixtures reduced exposure to those overfit domains.
- **Exogenous gap ($\Delta=1.26$).** CommonCrawl became the held-out target while related domains remained available for training. Target perplexity fell from 49.19 under Uniform to roughly 40–41 under optimized mixtures.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*The same optimizer family produces negligible final gains in the approximately matched regime and persistent gains when the task contains a correctable endogenous or exogenous gap.*

Selection provides a paired control. We kept the model, 5B-token [OpenWebMath](https://arxiv.org/abs/2310.06786) candidate pool, 60% keep ratio, selectors, and training protocol fixed. We changed only the target used for selection and evaluation.

- With an in-distribution OpenWebMath target, $\Delta=0.016$ and no selector consistently beat full-pool continual training.
- With a shifted [MetaMath](https://arxiv.org/abs/2309.12284)–[MAmmoTH](https://arxiv.org/abs/2309.05653) target, $\Delta=0.549$. Target-aware [RHO-1](https://arxiv.org/abs/2404.07965) and [BLADE](https://arxiv.org/abs/2606.18650) could now correct a visible mismatch; BLADE raised average benchmark accuracy from 17.21 to 31.31 while retaining 60% of the candidate tokens.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/selection-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Target access alone is not enough: the same target-aware selectors become useful when the target exposes a mismatch that the candidate pool and selection rule can correct.*

## A different order of operations

The usual workflow is:

1. invent a data-utility scoring rule;
2. select or reweight data;
3. train a model;
4. hope the target metric improves.

I think a more informative protocol is:

1. **Define the target.** What distribution and metrics represent the capability we actually care about?
2. **Run the uncurated reference and diagnose the task.** Estimate $\Delta$ with a small proxy when possible, remembering that a near-zero result is not proof of distributional equality.
3. **Check correctability.** Does the candidate pool contain the missing modes, and can mixture or selection expose them?
4. **Optimize the data only after specifying the claim.** Are we trying to save compute, move the attainable endpoint, or both?
5. **Validate accordingly.** For acceleration, report tokens, FLOPs, or wall-clock time at matched performance. For advancement, increase the budget and test whether the advantage persists.

This protocol makes the distinction explicit. Compute saving is useful and should be reported as such. But the strongest data methods do not merely help an undertrained model look better at an early checkpoint. They change the model that becomes attainable after the available budget has been spent.

That is the outcome I care about most. **Data curation is most consequential when it changes the attainable model, not merely the speed at which an undertrained model approaches it.**

And this leads to a deeper question. If the value of curation depends on a target and a correctable mismatch, can a sample be called “high quality” before either is specified?

That is the subject of the [next essay]({{ '/blog/2026/what-makes-data-good/' | relative_url }}).
