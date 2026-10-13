---
layout: post
published: false
title: "Data Curation Is Distribution Design: Beyond Quality and Diversity"
date: 2026-10-13
description: "Quality and diversity are useful proxies, but they do not define good data. A target-driven bilevel view unifies data mixture, selection, and synthesis as operators over the training distribution."
tags: llm data-curation bilevel-optimization data-selection data-synthesis
categories: research
thumbnail: assets/img/blog/data-curation/mixture-setup.png
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Draft: review the interpretation of external methods and add the Advancement paper link before publication. -->

What makes a training example “good”?

The standard answer in data engineering is some combination of **quality** and **diversity**. We remove malformed documents, assign quality scores, deduplicate near copies, balance domains, and try to cover a broad range of topics. These practices are useful, but they leave two questions unanswered:

- Quality relative to which reference?
- Diversity over which modes?

A polished web page can be useless for mathematical reasoning. A messy proof trace can be highly valuable for it. A corpus can be diverse in topic but narrow in the capabilities required by the target application. Conversely, adding more diversity can consume training budget on modes that do not matter for the objective.

The issue is not that quality and diversity are wrong. It is that they are **proxies or constraints**, not the final objective.

My preferred view is:

> **Data curation is the constrained design of a training distribution for a target objective.**

This perspective has gradually emerged across our work on domain mixture optimization ([TANDEM](https://openreview.net/forum?id=szBFUtBzWP)), token-level selection ([BLADE](https://arxiv.org/abs/2606.18650)), and our recent study of when curation provides lasting advancement rather than transient acceleration.

## Data utility is not an intrinsic property

Let \(p_{\mathrm{pool}}(x)\) denote the candidate-data distribution. A scoring or filtering rule does more than label samples as good or bad: it induces a new training distribution. If \(a_s(x)\) is the acceptance probability implied by a score \(s(x)\), then

$$
q_s(x)
=
\frac{p_{\mathrm{pool}}(x)a_s(x)}
{\int p_{\mathrm{pool}}(x')a_s(x')\,dx'}.
$$

The score matters only through the distribution it creates and the model learned from that distribution. This immediately makes “absolute quality” difficult to define. A score must encode some preference, and that preference comes from a reference corpus, a model, a judge, a validation set, or an implicit worldview embedded in the filtering rules.

Consider several influential approaches:

- **KenLM filtering in CCNet** trains a language model on Wikipedia and keeps documents with lower perplexity. Operationally, “quality” means being closer to the distribution represented by that language model.
- **DCLM-style classifiers** distinguish a positive corpus from random web data and retain high-scoring documents. The positive examples define the direction in which the web distribution is moved.
- **LLM-as-a-Judge filtering**, including instruction-data pipelines such as DEITA, turns the judge's preferences into an acceptance function.

These methods can be interpreted as **implicitly target-driven**. They work not because they discover an observer-independent property called quality, but because their reference happens to move the training distribution toward a useful target. If the reference and the desired capability diverge, the same filter can systematically remove useful data.

This interpretation is intentionally not a causal claim about why every method succeeds. It is a common language for asking what distribution each method induces and which target that distribution favors.

## The target-driven bilevel view

We can write the underlying problem in one line:

$$
\min_{q\in\mathcal Q}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(q)\right)
\quad\text{s.t.}\quad
\theta^*(q)
\in
\arg\min_\theta
\mathbb E_{x\sim q}\left[\ell(x;\theta)\right].
$$

The inner problem trains a model under distribution \(q\). The outer problem chooses \(q\) according to performance on a target. The feasible set \(\mathcal Q\) encodes what the data pipeline is actually allowed to do.

This formulation separates three objects that are often conflated:

1. **The target** defines the direction of improvement.
2. **The operator** defines which changes to the training distribution are feasible.
3. **The training budget** determines the cost of realizing the chosen distribution.

It also makes clear why target definition comes before scoring. Without \(\mathcal L_{\mathrm{target}}\), the outer problem is under-specified: many incompatible training distributions can all be called high quality.

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/mixture-setup.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*Mixture optimization as distribution design. The operation changes probability mass assigned to existing domains so that the effective training distribution better reflects the target.*

## Quality gives direction; diversity protects support

Where does diversity fit into this view?

A quality-oriented filter changes the *shape* of a distribution by moving probability mass toward preferred regions. Diversity is primarily about its *support*: whether the resulting data still covers the modes required by the target.

This distinction matters because aggressive filtering can improve average score while shrinking support. A classifier trained on a narrow positive set may produce a clean, coherent corpus that lacks rare domains, formats, languages, or reasoning patterns. Diversity constraints prevent the optimizer from collapsing onto a small set of easily recognized modes.

But “more diverse” is not automatically better. The useful requirement is closer to:

$$
\operatorname{supp}(p_{\mathrm{target}})
\subseteq
\operatorname{supp}(q),
$$

at the level of features or capabilities relevant to the task. In words: the training distribution should cover what the target may require.

This yields a cleaner division of labor:

> **Target alignment determines where probability mass should move.**  
> **Diversity prevents the support from collapsing while it moves.**

Quality and diversity remain important, but neither is meaningful enough to serve as the outer objective by itself.

## Three operators over the training distribution

Mixture, selection, and synthesis are often treated as separate research topics. Under the distribution-design view, they solve the same outer problem using different feasible operators.

| Operator | Decision variable | What it can change | Fundamental limit |
| --- | --- | --- | --- |
| Mixture | Domain weights | Probability mass across predefined groups | Cannot distinguish examples within a group |
| Selection | Sample/token weights or masks | Probability mass at fine granularity | Cannot create modes absent from the candidate pool |
| Synthesis | Generator, prompt, or sampling policy | Both probability mass and accessible support | Must control generation error and verify new data |

For mixture and selection, the induced distribution can be written abstractly as

$$
q(x)=\frac{r(x)p_{\mathrm{pool}}(x)}{Z},
$$

where \(r(x)\) is a domain weight, sample weight, or binary mask. Therefore,

$$
\operatorname{supp}(q)
\subseteq
\operatorname{supp}(p_{\mathrm{pool}}).
$$

No reweighting scheme can select an example that does not exist. If the target requires a reasoning pattern, tool trajectory, language, or business case absent from the pool, selection has reached its boundary. Synthesis is not merely another way to rank data; it expands the feasible set itself.

This distinction gives a practical rule:

- If the target pattern exists but is underrepresented, use **mixture or selection**.
- If the target pattern is missing, use **synthesis**.
- If synthetic data is noisy or too broad, apply **selection after synthesis**.

The three operators naturally form a pipeline rather than competing alternatives.

## From implicit targets to explicit optimization

Once a target set is explicit, we can optimize its performance directly instead of hoping that a generic score is a good proxy. This is the common thread between TANDEM and BLADE.

### TANDEM: domain-level distribution design

In data mixture optimization, the decision variable is a vector \(\alpha\) over domains. The ideal problem is

$$
\min_{\alpha\in\Delta^K}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(\alpha)\right),
\qquad
\theta^*(\alpha)
\in
\arg\min_\theta
\mathcal L_{\mathrm{train}}(\theta;\alpha).
$$

Solving this problem naively would require retraining a model for every candidate mixture. TANDEM turns the bilevel problem into a tractable penalty formulation and tracks it with two networks:

- a proxy model trained on the current training mixture;
- a reference model additionally shaped by target data.

The loss difference between the models indicates which domains benefit most from incorporating target information. Those domains receive more probability mass.

{% include figure.liquid path="assets/img/blog/data-curation/tandem-procedure.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*TANDEM updates domain ratios through twin proxy and target-informed reference models. The method optimizes target performance rather than a target-agnostic notion of domain quality.*

This formulation also revealed when mixture optimization has room to help. In abundant, single-pass pretraining, uniform allocation can be surprisingly competitive. Nontrivial mixtures become more valuable when limited domains are repeatedly sampled or when supervised fine-tuning creates generalization gaps. This observation later motivated our more explicit distinction between advancement and acceleration.

### BLADE: token-level distribution design

BLADE moves from domain weights to token-level decisions. Let \(\alpha_n\in[0,1]\) indicate whether token \(n\) contributes to the training loss, with a fixed keep ratio \(\gamma\). The bilevel objective is

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

Direct hypergradients involve an inverse Hessian and are impractical at LLM scale. BLADE instead uses a Hessian-free penalty reformulation. It maintains a training proxy \(u\) and a validation-informed reference \(w\), producing the token score

$$
s_n
=
\mathcal L^n_{\mathrm{train}}(u)
-
\mathcal L^n_{\mathrm{train}}(w).
$$

A Frank–Wolfe update over the fixed-budget feasible set yields a binary Top-\(\gamma N\) mask. In practice, this is applied within candidate batches, making online token selection possible without materializing scores for the entire corpus.

{% include figure.liquid path="assets/img/blog/data-curation/blade-procedure.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*BLADE instantiates the same outer objective at token granularity. A synchronized reference makes the target signal computationally usable during large-scale training.*

TANDEM and BLADE therefore differ in granularity and solver design, not in their ultimate objective. Both ask which feasible redistribution of training exposure most improves a target model.

## Before optimizing: is there an endpoint to improve?

The bilevel formulation tells us what to optimize, but it does not guarantee that a nontrivial optimum exists. If the uncurated training objective already matches the target objective, changing \(q\) may only improve finite-budget efficiency.

This is the motivation behind our distinction between **advancement** and **acceleration**, developed in the [previous essay]({{ '/blog/2026/advancement-or-acceleration/' | relative_url }}). A correctable train–target gap creates headroom for changing the endpoint. Without one, mixture and selection may still reach the same endpoint faster, but increasingly sophisticated allocation cannot manufacture a better asymptote.

The complete workflow is therefore:

1. Define the target and its acceptance criteria.
2. Train an uncurated baseline and diagnose the mismatch.
3. Determine whether the mismatch is represented in the candidate support.
4. Choose the least expensive operator capable of correcting it.
5. Retrain and evaluate on an independent target set across multiple budgets.

This order avoids a common failure mode: building a sophisticated scorer before establishing whether the task contains any correctable optimization headroom.

## Synthesis as support expansion

Data synthesis is the natural next step in this framework.

Suppose a generator \(G_\phi\) produces training data from prompts, specifications, environments, or model failures. The feasible training distribution now depends on both a generation policy \(\phi\) and a curation policy \(\alpha\):

$$
\min_{\phi,\alpha}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(\phi,\alpha)\right)
\quad\text{s.t.}\quad
\theta^*(\phi,\alpha)
\in
\arg\min_\theta
\mathbb E_{x\sim q_{\phi,\alpha}}[\ell(x;\theta)].
$$

The target should determine *what is missing*, not merely whether generated text looks plausible. A practical system can begin from failures on a gold target set, attribute those failures to missing skills or trajectories, synthesize candidate data directed at the deficiencies, and then select the candidates with the largest expected target utility.

This matters increasingly as the unit of data shifts from short documents or instruction pairs to long reasoning and agent trajectories. For such data, quality is not just fluency. It includes state transitions, tool feedback, delayed outcomes, process validity, and credit assignment. The same distribution-design principle still applies, but the operator now acts over tasks, trajectories, steps, and tokens.

## From static corpora to model-generated experience

Reinforcement learning makes the data problem more interesting because the training data is no longer fully exogenous. A behavior policy generates rollouts; rewards and verifiers evaluate them; selection, replay, and importance weighting determine which parts influence the next policy; that updated policy then changes the next round of data.

In this setting, there are at least two distinct distributional questions:

1. Does the generated experience cover and improve the downstream target capability?
2. Can trajectories produced by an older behavior policy be used reliably to update the current policy?

Our work on [Selective Importance Sampling (SIS)](https://arxiv.org/abs/2607.04728) addresses the second question. Standard importance sampling reweights off-policy data using the ratio between the current and behavior policies, but these ratios become unstable over long sequences. SIS instead performs a token-level acceptance–rejection test. Accepted tokens are certified as on-policy and receive unit importance weight; rejected tokens are not discarded, but retain the standard correction.

For this reason, SIS should not be described as ordinary data selection. It is better understood as a **stochastic distribution-conversion operator** applied between rollout and policy update. It nevertheless belongs to the same broader data-engineering picture: the effective distribution seen by optimization is jointly determined by how experience is generated, accepted, reweighted, and reused.

This suggests a richer data loop for reasoning models and agents:

$$
\text{target failures}
\rightarrow
\text{task synthesis}
\rightarrow
\text{policy rollouts}
\rightarrow
\text{trajectory/step/token curation}
\rightarrow
\text{policy update}
\rightarrow
\text{target evaluation}.
$$

Here the valuable data unit is no longer merely a document or instruction pair. It is a trajectory with state transitions, tool responses, intermediate decisions, and a delayed outcome. Length alone is not the point; the important property is structured dependency. Designing and curating such endogenous experience is, in my view, a natural next stage of target-driven data engineering.

## A different way to ask “what is good data?”

The useful question is not:

> Is this sample high quality?

It is:

> Under the target, candidate support, model-training process, and compute budget, how does assigning more probability to this sample change the model we obtain?

That question is harder. It is also operational: it tells us what to measure, why a reference model matters, when diversity is necessary, and when selection must give way to synthesis.

The view can be summarized in three lines:

> **Target defines the objective.**  
> **Gap determines the optimization headroom.**  
> **Operator determines whether that gap can be corrected.**

Quality and diversity remain indispensable tools. But the object we are ultimately engineering is the training distribution.
