---
layout: post
title: "There Is No Such Thing as Good Data—Without a Target"
date: 2026-10-13 11:00:00 +0800
description: "Quality and diversity are useful words, but incomplete objectives. A target-relative view changes what we score, select, and synthesize."
tags: llm data-curation data-quality diversity distribution-design
categories: research
thumbnail: assets/img/blog/data-curation/mixture-setup.png
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Series draft v2. Add the Advancement paper/arXiv link after it becomes public. -->

Imagine two training examples.

The first is a beautifully written encyclopedia article: clean grammar, reliable facts, coherent structure. The second is an ugly scratchpad from a failed geometry proof: abbreviations, backtracking, half-finished equations, and one crucial correction near the end.

Which one is better data?

For a general-purpose language model, perhaps the first. For a model that must recover from mistakes in mathematical reasoning, perhaps the second. For a product-description generator, probably neither.

Yet much of data engineering starts by asking whether a sample is “high quality,” as if quality were printed somewhere inside the sample. We build classifiers, reward models, heuristics, and LLM judges to uncover it. We then add *diversity* so that the high-quality set does not become too narrow.

This recipe often works. But the language hides a missing variable:

> **Quality according to whom—and diversity over what?**

My claim is not that quality and diversity are useless. It is that they are **underdefined without a target**. They are tools for designing a training distribution, not universal properties from which the right distribution automatically follows.

## Quality is a relationship disguised as an adjective

Suppose the raw pool follows $p_{\mathrm{pool}}(x)$, and a quality score induces an acceptance probability $a(x)$. After filtering, we train on

$$
q(x)
=
\frac{p_{\mathrm{pool}}(x)a(x)}
{\int p_{\mathrm{pool}}(x')a(x')\,dx'}.
$$

The scorer has not discovered a label attached to each sample by nature. It has changed the probability with which the learner sees different regions of the pool.

That change always has a direction.

A perplexity filter favors text that resembles the language model used to score it. A classifier favors text that resembles its positive examples. An LLM judge favors text that matches its rubric, preferences, and blind spots. Even a handcrafted rule such as “remove documents with too little punctuation” encodes a view of what useful text should look like.

Calling the output *higher quality* makes this operation sound neutral. It is not neutral. It is a distribution shift—sometimes an extremely useful one.

## One word, opposite recipes

The ambiguity becomes obvious when we list what the literature has called “high-quality” data.

Sometimes, **hard examples are good**:

- high-loss samples carry more learning signal;
- small-margin samples refine a decision boundary;
- frequently forgotten samples reveal unstable knowledge;
- examples with large gradients promise a larger parameter update.

Sometimes, **easy examples are good**:

- low-loss samples are less likely to be mislabeled;
- confident examples are safer for self-training;
- simple examples may support a curriculum;
- outliers and unusually hard samples may be noise rather than useful difficulty.

Sometimes, **preferred examples are good**: an LLM judge scores correctness, clarity, complexity, style, safety, or some weighted combination of them.

And sometimes, **influential examples are good**: influence functions, gradient matching, or hypergradient methods estimate how training on a sample changes performance on a particular validation target.

These definitions do not merely differ at the edges. They can rank the same sample in opposite directions. A high-loss example is valuable under an informativeness story and suspicious under a label-noise story. An unusual example improves coverage under one representation and becomes an outlier under another.

| Signal called “quality” | What it actually favors | Hidden question |
| --- | --- | --- |
| High loss / forgetting / small margin | Difficulty or current model uncertainty | Is difficulty learnable signal or noise? |
| Low loss / high confidence | Cleanliness and compatibility | Are easy examples informative enough? |
| LLM-as-a-Judge | Agreement with a model and rubric | Does the judge encode the desired capability? |
| Influence / hypergradient | Improvement on a validation target | Is the target representative and robust? |

The last family is conceptually different. Influence is not trying to recover context-free quality; it already asks a target-relative question. Its practical approximations may be imperfect, but the objective is explicit.

So “quality” is not one disputed quantity waiting for a better estimator. It is an overloaded interface covering several different desiderata: cleanliness, difficulty, informativeness, preference, and downstream utility. Before debating the best quality model, we should decide which of these problems we are solving.

## Successful filters often contain a hidden target

Consider three familiar families of methods.

**KenLM filtering**, as used in pipelines such as [CCNet](https://arxiv.org/abs/1911.00359), scores web documents with a language model trained on a curated reference such as Wikipedia. Low perplexity means “this document looks more like the reference.” The filter works by moving the web distribution toward that reference.

**DCLM-style classifiers** ([paper](https://arxiv.org/abs/2406.11794)) learn to distinguish a desirable positive corpus from broad web data. Again, the positive corpus supplies a direction. Change it, and the meaning of quality changes with it.

**LLM-as-a-Judge pipelines**, including instruction-data selection methods such as [DEITA](https://arxiv.org/abs/2312.15685), translate a model's preferences into an acceptance function. The judge may be excellent, but it is still a reference—not an oracle of context-free data value.

These methods can be reinterpreted as **implicitly target-driven**. Their “quality” signals are useful when the embedded reference points toward capabilities we care about. Their failures are also easier to understand in this language: if the reference is narrow, stale, stylistically biased, or simply misaligned with the downstream goal, a very accurate filter may efficiently move the data in the wrong direction.

This is a reinterpretation, not a claim that every reported gain has one proven causal explanation. But it forces a productive question:

> What distribution does this filter create, and which target would prefer that distribution?

## Diversity is a guardrail, not a compass

If quality moves probability mass, diversity tries to stop that mass from collapsing onto a few easy-to-recognize modes.

This matters. A narrow classifier can produce a dataset that is polished, coherent, and catastrophically incomplete. Rare languages disappear. Unusual formats disappear. Negative examples disappear. Messy reasoning traces disappear. The average quality score rises while the model loses coverage.

But “maximize diversity” does not solve the missing-target problem. Diversity over topics is not diversity over reasoning skills. Lexical diversity is not tool-use diversity. A million unrelated modes can be impressively diverse and completely irrelevant.

There is an even more basic problem: diversity exists only after choosing a representation and a notion of distance.

A geometric coreset may cover an embedding space. Cluster-based selection may sample near each centroid. A determinantal point process may reward pairwise separation. Submodular objectives may encourage diminishing returns. Each is perfectly reasonable—but each preserves a different geometry.

Change the encoder, distance metric, clustering granularity, or feature layer, and the “diverse” subset changes. Sampling one point from every topic cluster may preserve subject coverage while destroying coverage of difficulty. Sampling in gradient space may do the opposite. Even the cluster center is not universally representative: it favors what is typical inside the chosen geometry and may discard the rare boundary cases that matter most to a target.

Thus, diversity is not a scalar property of a dataset either. It is shorthand for *coverage under a chosen representation*. The missing question is which distinctions that representation must preserve.

What we usually need is closer to

$$
\operatorname{supp}(p_{\mathrm{target}})
\subseteq
\operatorname{supp}(q),
$$

where support should be understood at the level that matters: languages, skills, states, trajectories, failure modes, or other task-relevant features.

This gives quality and diversity different jobs:

> **A target tells us where probability mass should go.**  
> **Diversity keeps us from destroying the support we will later need.**

Diversity is indispensable because our targets and models are imperfectly specified. It protects options. But it cannot tell us which option deserves the next billion tokens.

## The same sample can change value without changing at all

Here is a useful sanity check for any intrinsic notion of data quality.

Take one sample $x$. Do not edit it. Now change only the target:

- from casual conversation to theorem proving;
- from English knowledge to multilingual customer support;
- from single-turn answers to long-horizon tool use.

Its value can change dramatically.

Now hold the target fixed and change the rest of the training pool. If the pool already contains a million near-duplicates of $x$, its marginal value falls. If $x$ is the only example covering an important failure mode, its value rises.

Nothing intrinsic to $x$ changed. What changed was its role in a system.

This is why per-sample scoring is such a lossy abstraction. A scalar score tries to compress at least four dependencies:

1. the target we want the model to serve;
2. the other data the model will see;
3. the training process that converts data into parameters;
4. the budget that limits how much exposure can be allocated.

Scores are often necessary engineering interfaces. We should simply remember what they approximate.

## From judging samples to designing distributions

The conceptual shift I find useful is small in wording but large in consequence:

> Do not ask whether a sample is good in isolation. Ask how changing its training probability changes the model obtained for a target.

In abstract form:

$$
\min_{q\in\mathcal Q}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(q)\right)
\quad\text{s.t.}\quad
\theta^*(q)
\in
\arg\min_\theta
\mathbb E_{x\sim q}[\ell(x;\theta)].
$$

The inner problem is ordinary model training. The outer problem chooses a feasible training distribution $q$ according to target performance.

This view makes three things explicit:

- **Target:** what behavior are we actually trying to improve?
- **Feasible data operations:** can we only rebalance domains, select existing samples, or generate new ones?
- **Budget:** how much data and compute can we spend?

It also separates the objective from its proxies. Quality scores, diversity constraints, deduplication rules, and safety filters may all be useful parts of $\mathcal Q$. None automatically replaces $\mathcal L_{\mathrm{target}}$.

## Why this is more than a philosophical complaint

“Good data depends on the task” can sound almost tautological. The useful part is what follows operationally.

### 1. Every scorer should declare its reference

Instead of reporting only how a quality model is trained, ask what implicit target it induces. What does a high score move the pool toward? Which useful modes receive low scores?

### 2. Diversity should be measured in target-relevant coordinates

Topic clusters may be appropriate for web pretraining. They may be inadequate for agent trajectories, where coverage of states, tools, recovery behaviors, and outcome types matters more.

### 3. Data methods should be evaluated across targets

A universal quality claim is strong. If a filter improves one reference-aligned benchmark and harms another, that is not an annoying edge case; it reveals the direction encoded by the method.

### 4. Selection should not be asked to invent missing support

If the target requires a mode absent from the candidate pool, no ranking rule can select it. The correct operation is synthesis, retrieval, or new data collection—not a more sophisticated score.

### 5. Improvement should be separated into acceleration and advancement

Even a target-aligned distribution may only improve learning speed when the baseline already has the right endpoint. In the [previous essay]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}), I discuss how a correctable train–target gap determines whether curation has room to create lasting advancement.

## What, then, is good data?

I would resist replacing “quality and diversity” with another two-word slogan. The whole point is that data value lives in a relationship.

A more complete question is:

> Under a target, a candidate pool, a training procedure, and a compute budget, which feasible change to the training distribution produces the model we prefer?

That question is harder than assigning a cleanliness score. It is also much closer to the decision we are actually making.

Quality still matters. Diversity still matters. But quality usually encodes a reference, and diversity protects a support. The object being engineered is the distribution presented to learning.

The [next essay]({{ '/blog/2026/from-target-to-operator/' | relative_url }}) turns this viewpoint into a technical map: mixture, selection, and synthesis as three operators over the same bilevel problem—and what changes when the data itself is generated by a policy.
