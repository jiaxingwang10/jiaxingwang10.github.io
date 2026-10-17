---
layout: post
title: "What Makes Data Good?"
date: 2026-10-13 11:00:00 +0800
description: "Quality and diversity are useful words, but incomplete objectives. A target-relative view changes what we score, select, and synthesize."
tags: llm data-curation data-quality diversity distribution-design
categories: research
thumbnail: assets/img/blog/data-curation/target-driven-data-pipeline.svg
series: designing-what-models-learn
series_title: "Designing What Models Learn"
series_order: 2
giscus_comments: false
related_posts: false
toc:
  sidebar: right
---

<!-- Series draft v3. Add the Advancement paper/arXiv link after it becomes public. -->

<p class="lead text-muted">Quality, Diversity, and the Missing Target</p>

> **Designing What Models Learn · Part 2 of 3**<br>
> *Essays on targets, training distributions, and data operators*<br>
> [Advancement vs. Acceleration]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}) · **What Makes Data Good?** · [From Target to Operator]({{ '/blog/2026/from-target-to-operator/' | relative_url }})

Imagine two training examples.

The first is a beautifully written encyclopedia article: clean grammar, reliable facts, coherent structure. The second is an ugly scratchpad from a failed geometry proof: abbreviations, backtracking, half-finished equations, and one crucial correction near the end.

Which one is better data?

For a general-purpose language model, perhaps the first. For a model that must recover from mistakes in mathematical reasoning, perhaps the second. For a product-description generator, probably neither.

Yet much of data engineering starts by asking whether a sample is “high quality,” as if quality were printed somewhere inside the sample. We try to uncover it with hand-written rules such as [RefinedWeb](https://arxiv.org/abs/2306.01116), language-model perplexity as in [CCNet](https://arxiv.org/abs/1911.00359), classifiers as in [DataComp-LM](https://arxiv.org/abs/2406.11794), learned quality dimensions as in [QuRating](https://arxiv.org/abs/2402.09739), and LLM judges as in [AlpaGasus](https://arxiv.org/abs/2307.08701) and [DEITA](https://arxiv.org/abs/2312.15685). We then add *diversity* so that the high-quality set does not become too narrow.

This recipe often works. But the language hides two missing variables:

> **Quality according to whom—and diversity over what?**

Suppose the raw pool follows $p_{\mathrm{pool}}(x)$, and a quality score induces an acceptance probability $a(x)$. After filtering, we train on

$$
q(x)
=
\frac{p_{\mathrm{pool}}(x)a(x)}
{\int p_{\mathrm{pool}}(x')a(x')\,dx'}.
$$

Operationally, the scorer does not uncover a natural label called *quality*. It decides how frequently different parts of the pool will appear during training.

The direction comes from a reference:

- a perplexity filter favors text that resembles the language model and corpus used to construct it;
- a classifier favors text resembling its positive examples;
- an LLM judge favors text that satisfies its rubric as the judge understands it;
- a handcrafted rule favors documents with the surface properties encoded by the rule.

An LLM judge does not intentionally seek its blind spots. But outside its competence it may reward fluent mistakes, penalize unfamiliar valid formats, or miss a capability the target needs. The accepted distribution reflects both the rubric and the judge's limitations.

Calling the result *higher quality* makes the operation sound neutral. A more informative description is: the scorer has shaped the training distribution toward a particular reference.

## “High quality” has supported opposite recipes

The ambiguity becomes obvious when we list what the literature has called “high-quality” data.

Sometimes, **hard examples are valuable**. [Forgetting events](https://arxiv.org/abs/1812.05159), [GraNd, and EL2N](https://arxiv.org/abs/2107.07075) use instability, gradient magnitude, or early training dynamics to identify informative examples.

Sometimes, **easy examples are safer**. Low-loss or high-confidence samples may be less likely to be mislabeled, as in [Confident Learning](https://arxiv.org/abs/1911.00068), while easy-to-hard ordering underlies [curriculum learning](https://dl.acm.org/doi/10.1145/1553374.1553380). Here, unusual difficulty may signal noise.

Sometimes, **preferred examples are valuable**. [QuRating](https://arxiv.org/abs/2402.09739) scores several human-facing qualities; [AlpaGasus](https://arxiv.org/abs/2307.08701) filters instruction data with a strong LLM; [DEITA](https://arxiv.org/abs/2312.15685) combines complexity, quality, and diversity.

And sometimes, **target-influential examples are valuable**. [Influence functions](https://arxiv.org/abs/1703.04730) and methods such as [LESS](https://arxiv.org/abs/2402.04333) ask how a training example affects a specified validation target.

These definitions can rank the same sample in opposite directions: a high-loss example is informative under one story and suspicious under another.

| Signal called “quality” | What it favors | The unresolved question |
| --- | --- | --- |
| High loss, forgetting, large gradient | Difficulty or current model uncertainty | Is the difficulty learnable signal or noise? |
| Low loss or high confidence | Cleanliness and compatibility | Are easy examples informative enough? |
| LLM judgment | Agreement with a model and rubric | Does that rubric represent the desired capability? |
| Influence or target-gradient similarity | Improvement on a validation target | Is the target representative, and is the approximation reliable? |

Gradient matching deserves a separate note. It is not automatically an influence method. [GRAD-MATCH](https://arxiv.org/abs/2103.00123), for example, selects a subset whose gradient matches the gradient of a training or validation set. It becomes target-relative when the gradient being matched comes from a target validation set; matching the full training gradient instead is primarily a compression objective. The use of gradients alone does not determine which question a method answers.

There is no single winner here. The scores estimate different desiderata: cleanliness, difficulty, preference, training-set fidelity, or downstream utility.

## Successful filters reveal their reference

Several influential filtering pipelines become easier to interpret once we ask what reference supplies their direction.

[CCNet](https://arxiv.org/abs/1911.00359) uses a KenLM model trained on Wikipedia to score web documents. Low perplexity means that a document resembles the reference language distribution. The resulting filter moves a broad web crawl toward Wikipedia-like text.

[DataComp-LM](https://arxiv.org/abs/2406.11794) makes this dependence especially visible. Its fastText filter contrasts a positive corpus with broad web data. Changing that positive corpus—from Wikipedia or OpenWebText to OpenHermes plus highly rated ExplainLikeImFive answers—changes what the classifier seeks; the latter produced the strongest filter in its ablation.

LLM-based pipelines expose the reference through a rubric. [QuRating](https://arxiv.org/abs/2402.09739), [AlpaGasus](https://arxiv.org/abs/2307.08701), and [DEITA](https://arxiv.org/abs/2312.15685) turn different prompts, judges, and criteria into different accepted distributions.

This gives us two concrete questions for any filter:

1. **What reference determines its acceptance rule?**
2. **Does moving the pool toward that reference improve the target we care about?**

The first identifies the direction of the shift; the second asks whether that direction is useful. Calling both “quality” obscures the distinction.

## A preference over data is not yet an objective for the model

A quality score tells us which examples should receive more probability mass. It does not, by itself, specify what shaping the training distribution in that direction should ultimately improve in the resulting model.

These are different objects. A scorer defines a preference over data; a target objective defines a preference over trained models. The connection between them runs through training—and that connection is exactly what many quality-based accounts leave implicit.

A Wikipedia-trained language model may prefer fluent encyclopedic text. A classifier trained on a curated positive corpus may prefer data resembling that corpus. An LLM judge may prefer responses that satisfy its rubric. Each supplies a potentially useful direction for moving probability mass. But none of these preferences alone guarantees that the resulting model will improve on the downstream distribution, capability, or evaluation that we actually care about.

> **A quality score says where to move probability mass. A target objective says how to judge the model that arrives after training.**

There is a second omission. Most quality scores rank examples one at a time. If ten thousand nearly identical documents all look excellent to the scorer, all ten thousand may receive high scores. The score expresses a preference, but it does not ask how much of that preference is already represented by the rest of the selected data.

## Diversity is the intuitive patch—but diversity in what?

This is where diversity enters naturally. Quality ranks examples individually; diversity discourages us from selecting many individually attractive but collectively redundant examples.

It is an important correction. Aggressive filtering can yield a selected dataset that is polished, coherent, and incomplete. A narrow classifier may *select against* rare languages, unusual formats, negative examples, or messy reasoning traces. It does not generate new samples; it forms the selected subset by assigning those regions lower acceptance probabilities. The average score can rise while useful coverage disappears.

But the patch introduces another choice: diversity is defined only after selecting a representation and a notion of distance.

A [geometric coreset](https://arxiv.org/abs/1708.00489) covers an embedding space; cluster-based selection such as the diversity component in [DEITA](https://arxiv.org/abs/2312.15685) spreads examples across neighborhoods; a [determinantal point process](https://arxiv.org/abs/1207.6083) rewards separation under its kernel. Each preserves a different geometry.

Change the encoder, feature layer, distance, or clustering granularity, and the “diverse” subset changes. Topic embeddings may preserve subject coverage but ignore reasoning difficulty. Gradient representations instead group samples by training effect: topically similar examples can separate when they induce different updates, and different topics can coincide when their updates agree.

In raw token space, a literal support statement is usually too fine-grained to be useful. Let $\phi(x)$ map an example to the features relevant to a task—perhaps language, skill, tool state, trajectory type, or failure mode. The operational coverage condition is better written as

$$
\operatorname{supp}\!\left(\phi_* p_{\mathrm{target}}\right)
\subseteq
\operatorname{supp}\!\left(\phi_* q\right),
$$

Here $\phi_*p$ is the pushforward distribution of task-relevant features under $\phi$. “Support” is not literally a list of skills; the feature map makes explicit which distinctions the coverage claim preserves.

A benchmark samples only some capabilities and failure modes, so preserving additional coverage hedges against what it failed to specify. But diversity alone cannot decide which region deserves the next billion tokens.

> **Diversity makes a pointwise preference aware of the selected set. It does not yet make that preference aware of the model the set will produce.**

Quality and diversity still have useful jobs. But their roles are different: **quality suggests a direction without fully specifying the destination; diversity preserves coverage without deciding where the model should go.** Neither, on its own, states how the shaped distribution should be judged through the model it produces.

## From scoring samples to optimizing a training distribution

Influence and validation-gradient methods move closer to the real question by introducing a target and a model state. But when their values are computed once and then used as fixed per-example scores, they remain local approximations. After the selected distribution changes, the model and its gradients change too; the original scores become stale. Joint interactions among the selected examples are represented only through the state at which the scores were measured.

One could repeatedly update the model, recompute the scores, and select again. But at that point, the procedure is better understood as an approximation to a larger optimization problem—not as the discovery of a permanent utility attached to each sample.

The cleaner object is the training distribution itself:

$$
q^*
=
\arg\min_{q\in\mathcal Q}
\mathcal L_{\mathrm{target}}\!\left(\theta^*(q)\right),
\qquad
\theta^*(q)=\operatorname{Train}(q).
$$

Here, $q$ describes the whole training distribution, $\theta^*(q)$ is the model produced by training on it, and $\mathcal L_{\mathrm{target}}$ says how models are compared. The outer optimization does not assign every example a context-free score and then patch redundancy afterward. It searches for weights jointly, through the model those weights produce.

If two groups of examples supply the same useful training effect, allocating more probability to either eventually offers less benefit than spending that mass elsewhere. What converges is not a universal data-utility score, but an allocation: a distribution whose components have been valued in the presence of one another.

Bilevel optimization did not invent target-aware data selection. Its value here is more basic: it provides a **common language for stating data problems**. Quality scores, diversity terms, influence estimates, and gradient similarity can all be understood as constraints, proxies, or approximations around this distribution-level objective.

The computational objection is real. Exact bilevel optimization may require differentiating through training or solving inverse-Hessian systems. In [BLADE](https://arxiv.org/abs/2606.18650), we use a Hessian-free penalized objective to recover a scalable excess-loss-style selection rule with a synchronized reference model. The objective is classical; the contribution is bridging it to a procedure that can be deployed at LLM scale.

## Put the operator and budget inside the problem

The compact formulation above still hides two practical constraints: which data transformations are allowed, and how much data or compute is available.

Let $O$ denote an operator, $B$ a budget, and $\mathcal Q_O(B;p_{\mathrm{pool}})$ the set of training distributions reachable from the original pool using that operator under that budget. Then the outer problem is

$$
\min_{q\in\mathcal Q_O(B;p_{\mathrm{pool}})}
\mathcal L_{\mathrm{target}}\!\left(\theta_B^*(q)\right).
$$

Now the missing pieces are visible:

- **Mixture:** $O$ can redistribute probability mass among existing domains.
- **Selection:** $O$ can retain, discard, or change the sampling probabilities of examples already in the candidate pool.
- **Synthesis:** $O$ can create samples and therefore enlarge the reachable support.
- **Budget:** $B$ constrains retained tokens, generated samples, training steps, or total compute; it restricts both $\mathcal Q_O$ and the inner training solution $\theta_B^*$.

Quality, diversity, deduplication, and safety can constrain $\mathcal Q_O$ or proxy the outer objective. They do not specify $\mathcal L_{\mathrm{target}}$: the behavior by which the trained model is ultimately judged.

{% include figure.liquid loading="lazy" path="assets/img/blog/data-curation/target-driven-data-pipeline.svg" class="img-fluid rounded z-depth-1" zoomable=true alt="Designing What Models Learn: a target guides data intervention over a raw pool; the operator and budget define a reachable distribution, which is trained and evaluated on the target." %}

*The formulation separates four things that quality and diversity often blur together: the target, the reachable training distributions, the training process, and evaluation of the resulting model.*

Selection cannot recover a target feature absent from the candidate pool. If it is unreachable under $\mathcal Q_{\mathrm{select}}$, the remedy is retrieval, collection, or synthesis—not a cleverer ranking score.

## An operational change in workflow

“Good data depends on the task” is nearly tautological. The useful part is what it changes in practice.

### 1. Every scorer should declare its reference

What positive corpus, rubric, model, validation set, or representation gives the score its direction? Which useful modes does that reference systematically underrate?

### 2. Diversity should name its geometry

Topic diversity, lexical diversity, embedding coverage, and gradient diversity are not interchangeable. A diversity claim should state which representation is being covered and why that representation preserves distinctions needed by the target.

### 3. Test the direction, then choose the operator

Evaluate filters across targets: gains on one reference-aligned benchmark and losses on another reveal the direction they encode. Then choose an operator capable of the required change. Mixture changes domain mass, selection works within a pool, and synthesis adds missing support.

### 4. Evaluate the trained model, not only the selected dataset

Cleaner documents, higher judge scores, or broader embedding coverage are intermediate properties. The outer question is whether the resulting model improves on the declared target—and, as discussed in the [previous essay]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }}), whether that improvement is advancement or only acceleration.

## What, then, is good data?

I would not replace “quality and diversity” with another two-word slogan. Data value lives in a relationship.

A more complete question is:

> Under a target, a candidate pool, an allowed data operator, a training procedure, and a budget, which feasible change to the training distribution produces the model we prefer?

Quality can provide useful reference signals. Diversity can protect coverage. Influence and gradient methods can approximate local changes. But the object being engineered is the distribution presented to learning, and its value is measured through the model that distribution produces for a target.

The [next essay]({{ '/blog/2026/from-target-to-operator/' | relative_url }}) develops the operator side of this picture: mixture, selection, and synthesis as different feasible sets around the same target-driven objective—and what changes when the data itself is generated by a policy.
