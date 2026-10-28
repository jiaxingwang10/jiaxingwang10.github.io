---
layout: post
title: "What Makes Training Data Good? Quality, Diversity, and the Missing Target"
date: 2026-10-13 11:00:00 +0800
description: "Quality and diversity are useful words, but incomplete objectives. A target-relative view changes what we score, select, and synthesize."
tags: llm data-curation data-quality diversity distribution-design
categories: research
series: designing-what-models-learn
series_title: "Designing Training Data for Language Models"
series_order: 2
lang: en
translation_url: /blog/2026/what-makes-data-good-zh/
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

{% include essay-language-switch.liquid %}

> **Designing Training Data for Language Models · Part 2 of 3**<br>
> *When to optimize, what makes data good, and how to change it.*<br>
> [Advancement vs. Acceleration]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}) · **What Makes Training Data Good?** · [From Target to Operator]({{ '/blog/2026/from-target-to-operator/' | relative_url }})

Imagine two training examples.

The first is a beautifully written encyclopedia article: clean grammar, reliable facts, coherent structure. The second is an ugly scratchpad from a failed geometry proof: abbreviations, backtracking, half-finished equations, and one crucial correction near the end.

Which one is better data?

For a general-purpose language model, perhaps the first. For a model that must recover from mistakes in mathematical reasoning, perhaps the second. For a product-description generator, probably neither.

“Good data depends on the task” is not a surprising claim. The problem is that many pipelines cease to behave as if it were true once a proxy score is treated as an intrinsic property of the data. Much of data engineering starts by asking whether a sample is “high quality,” as if quality were printed somewhere inside it. We try to uncover that quality with [heuristic rules](https://arxiv.org/abs/2112.11446), [reference language models](https://arxiv.org/abs/1911.00359), [learned classifiers](https://arxiv.org/abs/2406.11794), and [LLM judges](https://arxiv.org/abs/2402.09739). We then add *diversity* so that the high-scoring set does not become too narrow.

This recipe often works. But the language hides two missing variables:

> **Quality according to whom—and diversity over what?**

## “Quality” is too vague to optimize

Suppose the raw pool follows $p_{\mathrm{pool}}(x)$, and a quality score induces a selection weight or acceptance probability $w(x)$. After filtering, we train on

$$
q(x)
=
\frac{p_{\mathrm{pool}}(x)w(x)}
{\int p_{\mathrm{pool}}(x')w(x')\,dx'}.
$$

The scorer has not discovered a label attached to each sample by nature. It has changed the probability with which the learner sees different regions of the pool. But the literature gives strikingly different answers about which examples should receive more of that mass.

This ambiguity is obvious in how quality scores treat "difficulty".

Sometimes, **hard examples are good**:

- [high-loss samples](https://arxiv.org/abs/1604.03540) carry more learning signal;
- [small-margin samples](https://openreview.net/forum?id=7oPAgqxNb20) refine a decision boundary;
- [frequently forgotten samples](https://arxiv.org/abs/1812.05159) reveal unstable knowledge;
- examples with [large gradients](https://arxiv.org/abs/2107.07075) promise a larger parameter update.

Sometimes, **easy examples are good**:

- [low-loss samples](https://arxiv.org/abs/1804.06872) are less likely to be mislabeled;
- [confident examples](https://arxiv.org/abs/2001.07685) are safer for self-training;
- [simple examples](https://dl.acm.org/doi/10.1145/1553374.1553380) may support a curriculum.

Both recipes can be reasonable, yet they push probability mass in opposite directions. The word *quality* has not resolved the disagreement; it has merely been attached to both.

<div style="color: var(--global-text-color-light); opacity: 0.52; font-size: 0.86rem; font-style: italic; line-height: 1.55;" markdown="1">
Connection to Part 1. Difficulty-based scores may improve efficiency or robustness, but without an external target signal they do not by themselves identify the direction of a train–target gap. In the language of the [previous essay]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}), they are more natural guides to acceleration than to stable advancement.
</div>

## Successful filters often implicitly encode a target distribution

Many successful filters do not measure quality in the abstract. They implicitly define a target distribution over data, then favor examples that receive higher probability or preference under that distribution.

[CCNet](https://arxiv.org/abs/1911.00359) uses a KenLM model trained on Wikipedia to score web documents. Low perplexity means that a document resembles the reference language distribution. The resulting filter moves a broad web crawl toward Wikipedia-like text.

[DataComp-LM](https://arxiv.org/abs/2406.11794) makes this dependence especially visible. Its fastText filter contrasts a positive corpus with broad web data. Changing that positive corpus—from Wikipedia or OpenWebText to OpenHermes plus highly rated ExplainLikeImFive answers—changes what the classifier seeks; the latter produced the strongest filter in its ablation.

LLM-based pipelines induce a preference distribution through a rubric-conditioned judge. [QuRating](https://arxiv.org/abs/2402.09739), [AlpaGasus](https://arxiv.org/abs/2307.08701), and [DEITA](https://arxiv.org/abs/2312.15685) turn different prompts, judges, and criteria into different accepted distributions.

Viewed this way, a Wikipedia-trained language model, a classifier trained on a preferred corpus, and a rubric-conditioned LLM judge are different ways of parameterizing the same object: an implicit target distribution over training examples. Hence these methods work not because they uncover universal quality, but because their proxy distributions point in a useful direction.

<div style="color: var(--global-text-color-light); opacity: 0.52; font-size: 0.86rem; font-style: italic; line-height: 1.55;" markdown="1">
Connection to Part 1. These filters are most likely to yield stable gains when the target distribution they encode is better aligned with the downstream target than the original training pool. In that case, filtering is not discovering universal quality; it is closing an existing train–target gap. Their success is therefore consistent with the gap view of advancement developed in the [previous essay]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}).
</div>

Nevertheless, **agreement with a proxy distribution is not evidence of downstream value**. The proxy must be aligned with the downstream target, and that alignment can only be validated through the model trained on the shifted distribution. Even when the direction is correct, most quality scoring remains pointwise: it ignores relationships among examples and can spend a finite data budget repeatedly selecting near-duplicates, leaving the resulting dataset too narrow.

## Diversity is the intuitive patch—but diversity in what?

Diversity is the natural patch for this pointwise failure. It makes selection aware of the set, not just each example in isolation, and can discourage a high-scoring subset from collapsing around redundant examples.

But like quality, diversity is a meta-property rather than a concrete selection criterion. It becomes computable only after choosing what counts as different. A [geometric coreset](https://arxiv.org/abs/1708.00489) covers an embedding space; cluster-based selection such as [DEITA](https://arxiv.org/abs/2312.15685) spreads examples across neighborhoods; a [determinantal point process](https://arxiv.org/abs/1207.6083) rewards separation under its kernel. Each can be principled once its geometry is fixed, but different geometries preserve different distinctions: topic embeddings may cover subjects while missing reasoning difficulty, whereas gradient representations group examples by their effect on training. A choice that works for one target need not transfer to another.

The ambiguity becomes larger when diversity is combined with quality. We can filter by quality and then diversify, select the highest-scoring example from each cluster, or optimize a weighted combination of the two. These procedures need not select the same data, and “quality plus diversity” provides no common scale or target-derived rule for choosing among them.

Quality can propose a direction, and diversity can guard against redundancy. But their combination neither supplies a target-derived trade-off nor evaluates the model that the chosen distribution will produce.

## From scoring samples to optimizing a training distribution

The direct formulation is bilevel: choose a training distribution by evaluating the model it produces on the downstream target.

$$
\min_{q\in\mathcal Q}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(q)\right)
\quad\text{s.t.}\quad
\theta^*(q)
\in
\arg\min_\theta
\mathbb E_{x\sim q}[\ell(x;\theta)].
$$

The inner problem maps a training distribution $q$ to a trained model $\theta^*(q)$. The outer problem compares candidate distributions by evaluating those models on the downstream target. The decision variable is the distribution as a whole, not a collection of permanent sample scores.

This makes the missing interaction explicit. The value of assigning more probability to one region depends on what else has already been selected and on how the learner responds to the resulting allocation. A pointwise quality score ignores this interaction. A diversity term can partially account for redundancy, but only under the geometry induced by the chosen representation. The bilevel objective instead evaluates the allocation as a whole through the model it produces.

## Reachability still matters

The bilevel objective tells us how to compare training distributions, but not which distributions are reachable. Selection can only reweight or remove examples already in a candidate pool. If a needed capability is absent, the remedy is retrieval, collection, or synthesis—not a better ranking score.

More generally, the feasible set $\mathcal Q$ is defined by the data operator and its budget. Mixture redistributes mass across predefined groups, selection acts within an existing pool, and synthesis can expand its support. The target determines how reachable distributions are evaluated; the operator and budget determine which distributions can be considered at all. The [next essay]({{ '/blog/2026/from-target-to-operator/' | relative_url }}) shows how mixture, selection, and synthesis instantiate different feasible sets within the same bilevel formulation—and how the resulting optimization problems can be approached.

## What, then, is good data?

Some decisions remain straightforward: remove invalid files, exact duplicates, or verified label errors. Beyond such hygiene, however, “good” is not a property of an example in isolation. It describes the role that example plays alongside the rest of the training distribution for a target.

I would not replace “quality and diversity” with another slogan. The more useful question is:

> Given a target, a training procedure, a budget, and a feasible family of training distributions $\mathcal Q$, which $q\in\mathcal Q$ produces the model we prefer?

Quality scores can supply useful signals, and diversity can protect coverage. But they are proxies or constraints within this decision, not target-independent definitions of training value.

Good data, then, is not a universal subset waiting to be discovered. It is a training distribution to be designed for a target.

{% include essay-footer.liquid citation_key='wang2026gooddata' %}
