---
layout: post
title: "From Target to Operator: Mixture, Selection, Synthesis, and RL Data"
date: 2026-10-13 10:00:00 +0800
description: "A bilevel view of data engineering: targets define the objective, operators define what distributions are reachable, and gaps determine whether optimization can help."
tags: llm data-curation bilevel-optimization data-selection data-synthesis reinforcement-learning
categories: research
thumbnail: assets/img/blog/data-curation/blade-procedure.png
series: designing-what-models-learn
series_title: "Designing What Models Learn"
series_order: 3
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Series draft v2. Add the Advancement paper/arXiv link after it becomes public. -->

> **Designing What Models Learn · Part 3 of 3**<br>
> *Essays on targets, training distributions, and data operators*<br>
> [Advancement vs. Acceleration]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}) · [What Makes Data Good?]({{ '/blog/2026/what-makes-data-good/' | relative_url }}) · **From Target to Operator**

Suppose you run a warehouse full of training data.

You can change how many boxes arrive from each supplier. You can open the boxes and keep only particular items. Or you can manufacture items that were never in the warehouse at all.

These sound like three different businesses. In machine learning, we call them **data mixture**, **data selection**, and **data synthesis**—and they are usually studied by different communities with different terminology.

I think they are three operators over the same object: the training distribution.

This viewpoint gives us a compact map:

> **The target defines where we want to go.**  
> **The gap determines whether there is somewhere better to go.**  
> **The operator determines whether we can get there.**

The rest is optimization.

## One outer problem, many data pipelines

Let $q$ denote the effective distribution seen during training. A target-driven data problem can be written as

$$
\min_{q\in\mathcal Q}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(q)\right)
\quad\text{s.t.}\quad
\theta^*(q)
\in
\arg\min_\theta
\mathbb E_{x\sim q}[\ell(x;\theta)].
$$

The inner problem maps data to a trained model. The outer problem chooses data according to what that model does on a target. This is why the problem is naturally bilevel: data is not evaluated by appearance alone, but by the learner it produces.

The interesting object is the feasible set $\mathcal Q$. It records what the pipeline is actually capable of changing.

| Operator | Decision variable | What it can change | What it cannot do |
| --- | --- | --- | --- |
| Mixture | Domain weights | Exposure across predefined groups | Distinguish samples inside a group |
| Selection | Sample/token weights or masks | Fine-grained exposure within a pool | Create a missing mode |
| Synthesis | Generator, prompt, task, or rollout policy | Exposure and accessible support | Guarantee correctness of new data |

Mixture and selection reallocate probability mass. Synthesis can enlarge the space over which that mass is allocated.

That difference sounds obvious. It has surprisingly strong consequences.

## Reweighting has a hard boundary

For mixture and selection, the induced distribution can be written abstractly as

$$
q(x)=\frac{r(x)p_{\mathrm{pool}}(x)}{Z},
$$

where $r(x)$ is a domain weight, sample weight, or binary mask. Therefore,

$$
\operatorname{supp}(q)
\subseteq
\operatorname{supp}(p_{\mathrm{pool}}).
$$

No amount of clever reweighting can select an example that does not exist.

If the target skill is present but rare, mixture or selection can expose it more often. If the target requires a tool trajectory, language, failure-recovery pattern, or reasoning strategy absent from the pool, selection has reached its boundary. At that point, repeatedly improving the ranker is like reorganizing a library in search of a book nobody bought.

This gives a basic operator-selection rule:

- **Underrepresented but present:** mixture or selection.
- **Missing from the support:** collect or synthesize.
- **Synthesized but noisy:** synthesize first, then select or verify.

## Before choosing an operator, check the gap

Even a powerful operator is unnecessary when the baseline training objective already matches the target.

This is the distinction between [advancement and acceleration]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}). With no correctable train–target gap, reallocating existing support may make learning faster, but it should not move the endpoint. A nonzero gap creates headroom; the feasible operator decides whether that headroom is reachable.

That suggests the following order:

1. define a target distribution and acceptance criteria;
2. train an uncurated baseline;
3. diagnose the observed train–target mismatch;
4. locate the missing or underweighted modes;
5. choose the least expensive operator capable of correcting them.

This order is easy to invert. It is tempting to build a sophisticated data scorer first because scoring is concrete. But a perfect optimizer cannot rescue an ill-posed outer objective or an unreachable target.

## TANDEM: change mass at the domain level

In data-mixture optimization, the decision is a vector $\alpha$ over $K$ domains:

$$
\min_{\alpha\in\Delta^K}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(\alpha)\right),
\qquad
\theta^*(\alpha)
\in
\arg\min_\theta
\mathcal L_{\mathrm{train}}(\theta;\alpha).
$$

The obvious implementation—train a fresh model for every candidate mixture—is hopelessly expensive. [TANDEM](https://openreview.net/forum?id=szBFUtBzWP) uses a penalty reformulation and two coupled networks instead:

- a proxy model trained on the current mixture;
- a reference model additionally informed by target data.

Their loss difference reveals which domains benefit most from target information. The outer update gives those domains more probability mass.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/tandem-procedure.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*TANDEM turns target performance into domain-allocation updates through coupled proxy and reference models.*

An unexpected lesson from this work was that uniform allocation can be annoyingly strong in abundant, single-pass pretraining. Nontrivial mixtures become more valuable when limited domains are repeatedly sampled or when supervised fine-tuning creates generalization gaps. That observation eventually led us to study the conditions for advancement directly.

## BLADE: change mass at the token level

Domain weights are coarse. A domain can contain both valuable and useless examples; an example can contain both valuable and useless tokens.

[BLADE](https://arxiv.org/abs/2606.18650) pushes the same outer objective to token granularity. Let $\alpha_n\in[0,1]$ indicate whether token $n$ contributes to training, under a fixed keep ratio $\gamma$:

$$
\min_{\alpha\in\mathcal A}
\mathcal L_{\mathrm{val}}\!\left(w(\alpha)\right)
\quad\text{s.t.}\quad
w(\alpha)
\in
\arg\min_w
\frac{1}{\gamma N}
\sum_{n=1}^N \alpha_n\mathcal L^n_{\mathrm{train}}(w).
$$

Exact bilevel hypergradients require inverse-Hessian information—an unattractive phrase even before “at LLM scale” is added. BLADE uses a Hessian-free penalty reformulation. A training proxy $u$ and a target-informed reference $w$ produce the token score

$$
s_n
=
\mathcal L^n_{\mathrm{train}}(u)
-
\mathcal L^n_{\mathrm{train}}(w).
$$

A Frank–Wolfe step over the budget constraint gives a binary Top-$\gamma N$ mask. Applied within candidate batches, the procedure performs online token selection without scoring and storing the entire corpus in advance.

{% include figure.liquid path="assets/img/blog/data-curation/blade-procedure.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*BLADE instantiates target-driven distribution design at token granularity. The coupled reference is a practical solver for the static bilevel objective, not a different objective.*

TANDEM and BLADE differ in granularity and numerical method. Their question is the same: which feasible redistribution of training exposure produces a model that performs better on the target?

## Synthesis: enlarge the feasible distribution

Selection assumes the missing value is hiding somewhere in the pool. Synthesis removes that assumption.

Let a generator $G_\phi$ produce candidates from prompts, specifications, environments, or observed failures. The outer problem now controls both generation and curation:

$$
\min_{\phi,\alpha}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(\phi,\alpha)\right)
\quad\text{s.t.}\quad
\theta^*(\phi,\alpha)
\in
\arg\min_\theta
\mathbb E_{x\sim q_{\phi,\alpha}}[\ell(x;\theta)].
$$

The formulation changes the synthesis question. Instead of asking “Can the generator produce plausible data?”, ask:

> Which missing target behaviors should be generated, and does training on them repair the measured failure?

A practical loop might look like this:

$$
\text{target failures}
\rightarrow
\text{failure attribution}
\rightarrow
\text{directed synthesis}
\rightarrow
\text{selection / verification}
\rightarrow
\text{training}
\rightarrow
\text{target evaluation}.
$$

This is more demanding than bulk self-instruction. It treats generation as an intervention on the support, followed by an empirical test of whether that intervention closes the gap.

## The data unit is becoming a trajectory

Documents and instruction–response pairs are convenient, mostly static units. Reasoning models and agents create a messier object: a trajectory.

A useful trajectory may include a task specification, hidden environment state, intermediate reasoning, tool calls, tool responses, recovery from errors, and a delayed outcome. “Quality” is no longer captured by fluency or even final-answer correctness. The path matters.

Reinforcement learning also makes the data endogenous. The policy generates rollouts; verifiers and rewards evaluate them; selection, replay, and importance weighting determine which parts affect the next update; the new policy then changes the next round of data.

There are at least two distributional questions:

1. Does the generated experience cover the downstream target capability?
2. Can experience from an older behavior policy be used reliably to update the current policy?

Our work on [Selective Importance Sampling (SIS)](https://arxiv.org/abs/2607.04728) addresses the second. Standard importance sampling reweights off-policy data with ratios between the current and behavior policies, but sequence-level products become unstable over long horizons. SIS uses a token-level acceptance–rejection test. Accepted tokens are converted to on-policy samples with unit weight; rejected tokens are retained with the usual importance correction.

SIS is therefore not ordinary data selection: rejection does not mean deletion. I find it more useful to view SIS as a **stochastic distribution-conversion operator** between rollout and policy update.

This expands the data-engineering map. An RL pipeline does not merely choose a static dataset. It designs how experience is generated, verified, accepted, reweighted, reused, and regenerated.

## A research program hiding inside a data pipeline

The unified view does not imply that one algorithm should solve everything. Quite the opposite: once the target and feasible set are explicit, different technical questions become easier to separate.

- **Diagnosis:** Is the observed gap large enough to justify intervention?
- **Attribution:** Which missing or overrepresented modes create it?
- **Operation:** Should we rebalance, select, synthesize, or convert experience?
- **Optimization:** How can the outer decision be approximated without retraining a model for every candidate?
- **Verification:** Did the gain persist with budget, and did another target regress?

For simple corpora, the operator may be a vector of domain weights. For web-scale filtering, it may be a token mask. For agents, it may be a task generator plus a trajectory verifier plus a replay rule. The mathematical skeleton can remain the same while the unit of intervention becomes richer.

## The map I use

When I face a new data problem, I now try to fill in four blanks before thinking about a scorer:

1. **Target:** What behavior will decide whether the model is better?
2. **Gap:** What does the baseline fail to learn from its current distribution?
3. **Support:** Is the missing behavior already present in the available data?
4. **Operator:** What is the cheapest allowed intervention that can expose or create it?

This does not make data engineering easy. It makes the source of difficulty visible.

Mixture, selection, and synthesis are not competing slogans. They are different moves in distribution space. Their usefulness is determined jointly by where the target lies, where the baseline starts, and what regions the operator can reach.

Or, in three lines:

> **Target defines the objective.**  
> **Gap defines the headroom.**  
> **Operator defines the reachable improvement.**
