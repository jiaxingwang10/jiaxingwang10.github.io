---
layout: post
published: true
title: "消失的 3.5 点优势：数据优化是在加速，还是在改善终点？"
permalink: /blog/2026/the-3-point-win-that-disappeared-zh/
date: 2026-10-13 12:00:00 +0800
description: "一个数据配方在固定预算下领先，并不意味着它能带来更好的终点。如何区分数据优化带来的是加速，还是持续的能力提升？"
lang: zh-CN
translation_url: /blog/2026/the-3-point-win-that-disappeared/
series: designing-what-models-learn
series_title: "语言模型的训练数据设计"
series_order: 1
giscus_comments: false
related_posts: false
toc:
  sidebar: right
_styles: |
  .essay-chinese {
    line-height: 1.9;
    overflow-wrap: break-word;
  }
  .essay-chinese h2 {
    margin-top: 2.25rem;
    line-height: 1.5;
  }
  .essay-chinese h3 {
    line-height: 1.5;
  }
---

{% include essay-language-switch.liquid %}

<div class="essay-chinese" lang="zh-CN" markdown="1">

本文为[英文原文]({{ '/blog/2026/the-3-point-win-that-disappeared/' | relative_url }})的中文版本，由 ChatGPT 辅助翻译。

> **语言模型的训练数据设计 · 三篇系列之一**<br>
> *什么时候值得优化，什么是好数据，以及如何改变数据。*<br>
> **加速与终点改善** · [什么是好数据？（英文）]({{ '/blog/2026/what-makes-data-good/' | relative_url }}) · [从目标到操作（英文）]({{ '/blog/2026/from-target-to-operator/' | relative_url }})

我们一直在尝试通过优化训练数据的配比来提升语言模型的能力。想法很直接：如果能更合理地把 token 预算分配到不同领域，是不是就能训练出更好的模型？

在一个实验中，我们沿用 [Aioli](https://arxiv.org/abs/2411.05735) 的设置，在 [SlimPajama](https://www.cerebras.net/blog/slimpajama-a-627b-token-cleaned-and-deduplicated-version-of-redpajama) 的七个领域上搜索数据配比：ArXiv、Books、C4、CommonCrawl、GitHub、StackExchange 和 Wikipedia。我们尝试了 [RegMix](https://arxiv.org/abs/2407.01492)：先用不同配比训练一批小型代理模型，再拟合它们的实验结果，为更大的训练推荐配比。得到配比后，我们从头训练一个新模型。

训练到 2,000 步时，这个模型比均匀配比的基线低了 **3.5 点平均困惑度**。看上去，这正是我们期待的结果。如果像许多固定预算的比较一样，到这里就停下来，我们大概会认为这是一次很有说服力的成功。

但我们让这次训练继续进行，使用更多数据。

优势越来越小。再小一些。到了最后，它消失了。

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-acceleration.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*在 2,000 步时，RegMix 的困惑度比 Uniform 低 3.5 点；到了最后一个 checkpoint，它反而略差。关键不是两次训练得到了完全相同的结果，而是 RegMix 的领先没有保持下来。*

这次消失的优势让我开始思考一个问题，后来它成为了这项工作的核心：

> **当数据优化让语言模型表现更好时，它改变的是模型最终能学到什么，还是到达同一水平的速度？**

实验中的提升是真实的，但它的含义比早期 checkpoint 看起来要窄：RegMix 更早达到了某个性能水平，却没有在这次实验中带来更好的最终结果。

关于数据优化的许多讨论，都把下面两件事统称为“提升”：

> **加速（acceleration）**：用更少的数据达到同一性能水平。  
> **终点改善（advancement）**：提高模型最终能够达到的性能水平。

加速本身就有价值：如果达到同样的能力只需要少得多的 token，这当然是有意义的收益。但这里我想追问的是另一个问题：**改变训练数据，能否改善模型可达到的终点？** 开头那个逐渐消失的优势，让我觉得有必要认真区分这两件事。

## 终点，以及到达终点的路

用 $\mathcal E_\alpha(w)$ 表示：采用数据策略 $w$、使用数据预算 $\alpha$ 训练后，模型在目标上的表现。这里的 $w$ 可以是领域配比、样本选择，也可以是 token mask。为了方便写不等式，下文统一把 $\mathcal E$ 写成越小越好的指标，例如 loss、错误率，或准确率的相反数；如果直接使用准确率，只需把不等号反过来。

终点定义为

$$
\mathcal E_\infty(w)=\lim_{\alpha\rightarrow\infty}\mathcal E_\alpha(w).
$$

如果

$$
\mathcal E_\infty(w)<\mathcal E_\infty(w_0),
$$

我们说策略 $w$ 带来了**终点改善**。其中，$w_0$ 表示未经数据优化的基线策略。

对于某个目标性能水平 $\varepsilon$，用 $\alpha_w(\varepsilon)$ 表示策略 $w$ 达到它所需的最小预算。如果

$$
\alpha_w(\varepsilon)<\alpha_{w_0}(\varepsilon),
$$

我们说它带来了**加速**。

加速改变的是到达同一水平所需的数据量；终点改善改变的是最终能达到的水平。只看一个 checkpoint，无法区分二者。

## 数据优化凭什么改变终点？

假设有一个装着红蓝弹珠的大罐子，我们要在另一个罐子里复现相同的红蓝比例。更聪明的采样方法，可能让我们更快估准这个比例。但如果普通采样本来就对应正确的分布，那么在极限下，重新加权也不可能给出一个“比正确答案更正确”的答案。

数据优化也受到类似的约束。把它写成一个双层优化问题：

$$
\min_w \; \mathcal L_{\mathrm{target}}\!\left(\theta^*(w)\right)
\quad\text{s.t.}\quad
\theta^*(w)\in\arg\min_\theta \mathcal L_{\mathrm{train}}(\theta;w).
$$

内层优化模型参数 $\theta$；外层优化数据策略 $w$，通过它改变训练得到的模型 $\theta^*(w)$。但如果基线的训练目标已经与目标任务的评价目标**一致**，更好的终点还能从哪里来？

### 一个很短的证明

令 $w_0$ 表示基线策略。关键假设是：训练与目标**在总体分布层面没有差距**，即

$$
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0)
=
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta)
\qquad \text{for every }\theta.
$$

这里的“一致”，是指固定 $w_0$ 后，训练 loss 和目标 loss 是关于 $\theta$ 的同一个函数，而不只是在某个 checkpoint 上数值相等。上标 $\mathrm{pop}$ 表示总体风险（population loss），也就是对底层数据分布取期望。在理想化的大数据极限 $\alpha\to\infty$ 下，收敛后的训练求解的是这个目标。

当内层优化收敛时，基线解

$$
\theta^*(w_0)\in\arg\min_\theta
\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w_0)
$$

也全局最小化了目标 loss——因为在 $w_0$ 下，训练 loss 和目标 loss 本来就是同一个函数。对于其他数据策略 $w$，训练得到的 $\theta^*(w)$ 也仍然只是同一模型类中的另一个模型。因此

$$
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}\!\left(\theta^*(w_0)\right)
=
\min_\theta \mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta)
\leq
\mathcal L_{\mathrm{target}}^{\mathrm{pop}}\!\left(\theta^*(w)\right).
$$

约束就在这里：**当基线已经最小化目标总体风险时，重新分配数据不可能得到严格更好的目标终点。** 这个结论以模型类固定、内层优化达到上述最小值为前提。

证明很简单，但它的含义在实践中容易被忽略。一个更好的配比或子集在有限预算下领先时，我们很容易把它理解成终点也会更好。然而，如果总体分布层面没有可纠正的差距，看到的收益就可能只是加速。

<div style="color: var(--global-text-color-light); opacity: 0.52; font-size: 0.86rem; line-height: 1.55;" markdown="1">
**以数据配比为例。** 对 $K$ 个领域，$w_k$ 是领域 $k$ 在训练中的比例，$\widehat w_k$ 是它在目标评价中的固定权重。两组权重都非负，且各自加和为一。总体目标写成

$$
\begin{aligned}
&\mathcal L_{\mathrm{train}}^{\mathrm{pop}}(\theta;w)
=\sum_{k=1}^K w_k\mathcal L_{\mathrm{train}}^{k,\mathrm{pop}}(\theta),\\
&\mathcal L_{\mathrm{target}}^{\mathrm{pop}}(\theta)
=\sum_{k=1}^K\widehat w_k\mathcal L_{\mathrm{target}}^{k,\mathrm{pop}}(\theta).
\end{aligned}
$$

我们优化的是训练配比 $w$。如果每个领域内训练与目标的数据分布相同，而且 $w_0=\widehat w$，两个总体目标就完全一致，对应上面的无差距条件。

当训练与评价数据从同一底层分布独立同分布采样时，数据越充足，经验目标就越接近同一个期望 loss。这是数据充足、单遍训练场景背后所对应的总体分布图景。

开头的七领域 SlimPajama 实验采用均匀的目标权重，$\widehat w_k=1/K=1/7$。均匀配比基线因此满足 $w_{0,k}=\widehat w_k$。在各领域内训练与评价来自相同底层分布的条件下，这个基线在总体终点上已经最优；学习曲线早期的领先则可以由加速解释。即使不同领域的学习难度不同，这一点也不会改变：给困难领域更多权重，可能改善有限预算下的表现，但难度差异本身并不会在已匹配的目标上创造更好的终点。
</div>

这些结果将在我们的论文 **Advancement or Acceleration? How Data Curation Improves LLM Training** 中展开。

<!-- Add the public arXiv link to the paper title above once available. -->

> **持续的终点改善，需要可纠正的训练—目标差距（train–target gap）。**

下文简称它为 gap。存在 gap 意味着可能有改善空间，却不保证我们能利用这个空间。现有数据，以及我们能够改变数据使用方式的手段，必须允许我们缩小 gap。这就是这里“可纠正”的含义。

## 在模型中看见加速与终点改善

为了把区别看清楚，我们构造了 teacher–student 感知机模型：teacher 生成标签，student 从经过配比或筛选的数据中学习。相关分析框架可以参考 [Loureiro 等人（2021）](https://arxiv.org/abs/2102.08127) 关于学习曲线的研究，以及 [Sorscher 等人（2022）](https://arxiv.org/abs/2206.14486) 关于数据剪枝的研究。这类模型让我们既能研究有限预算下的学习，也能研究 $\alpha\to\infty$ 的终点，区分哪些收益会消失、哪些会持续。技术推导留在论文中。

下面的图按行读：**上排是数据配比（mixture），下排是数据选择（selection）**。左列介绍设置，中列展示无 gap 时的加速，右列展示有 gap 时的变化。这里 $\alpha=P/N$ 表示每个参数对应的样本数，也就是归一化的数据预算。

**Mixture：重新加权可以在无 gap 时加速，也可以通过纠正 gap 改善终点。** 为了展示加速，我们在上排的设置中让领域 2 比领域 1 更难学习，具体构造见论文。$w_1$ 表示领域 1 的训练比例；在图 (b) 中，它的目标权重是 $\widehat w_1=0.5$。均匀基线的训练比例与目标权重一致，没有 gap。此时，向困难领域多分配一些数据，可以在小预算下带来收益，但随着预算增大，优势会消失。当目标权重改变时，最优配比也随之改变：图 (c) 展示了在一个大但有限的预算下，匹配目标配比最优。论文进一步说明，即使领域难度不同，这个最优性在 $\alpha\to\infty$ 时仍然成立。

**Selection：剪枝可以在无 gap 时加速，也可以通过纠正 gap 改善终点。** 在下排，我们用两个线性感知机 teacher 分别构造训练和目标的标签规则。每个 teacher 都可由决策边界的法向量描述：训练对应 $B$，目标对应 $B_{\mathrm{tar}}$；二者夹角 $\phi$ 控制训练—目标差距，如图 (d)。$f$ 是保留比例，$f=1$ 表示全部保留。筛选准则偏好靠近目标决策边界的样本，并惩罚训练标签与目标 teacher 不一致的样本。没有 gap 时，适度剪枝可以用更少数据保持准确率，却不能改善终点，如图 (e)。存在 gap 时，筛选在“样本更少”和“更贴近目标”之间作出权衡。图 (f) 表明，对齐目标的收益可以超过减少样本的代价；论文进一步说明，这种收益可以保持到 $\alpha\to\infty$。

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/advancement-theory-map.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*上排的目标 loss 越低越好，下排的目标准确率越高越好。Selection 的横轴从左侧的全部保留，走向右侧的更强剪枝。实线是理论预测，圆点是 $N=200$ 的模拟结果。图片保留英文标注，便于与论文对照。*

## 昂贵搜索之前，先做一次诊断

总体分布层面的条件很清楚，但通常无法在真实语料上直接检验。因此，我们使用 **$\Delta$-test**，从一次普通的、未经数据优化的 baseline 训练中得到可观测的诊断信号。用 $\theta$ 表示这次训练的一个模型 checkpoint，无需等到训练收敛。

对于 $K$ 个领域的数据配比，定义

$$
\Delta
=
\sum_{k=1}^{K}
\left|
\widehat w_k\mathcal L_{\mathrm{target}}^k\!\left(\theta\right)
-
w_{0,k}\mathcal L_{\mathrm{train}}^k\!\left(\theta\right)
\right|.
$$

其中，$\widehat w_k$ 是领域 $k$ 在目标分布中的权重，$w_{0,k}$ 是它在基线训练中的比例。

对于 selection，候选数据来自单一数据池，诊断量简化为

$$
\Delta
=
\left|
\mathcal L_{\mathrm{target}}\!\left(\theta\right)
-
\mathcal L_{\mathrm{train}}\!\left(\theta\right)
\right|,
$$

这里基线策略 $w_0$ 对应全部保留。

在我们的实验中，$\Delta$-test 是一个有用的早期信号：明显非零的值对应持续收益的机会，而 $\Delta$ 接近零时，早期收益往往会随预算增加而消退。接近零提示改善空间可能有限，但不能排除所有训练—目标差距。另一方面，$\Delta$ 非零也不是终点改善的充分条件：能否纠正检测到的 gap，仍取决于现有数据和可用的数据策略。

## 改变 gap，观察同一批方法的表现

我们在 mixture 和 selection 实验中检验两个问题：数据优化的收益是否取决于训练—目标差距？$\Delta$-test 能否在昂贵搜索之前识别值得尝试的任务？为此，我们改变 gap，同时让方法和训练设置尽量保持一致。

下面的实验只追踪**已测试训练预算内**的收益；无限数据下的结论来自前面的理论分析。报告的 $\Delta$ 都在基线训练预算过半时测量，而不是等训练完全收敛。实际使用时，也可以先用小型 proxy 模型诊断，再决定是否进行更大规模的实验，降低额外计算成本。

对于 mixture，我们比较了 Uniform，以及 [Skill-It](https://arxiv.org/abs/2307.14430)、[RegMix](https://arxiv.org/abs/2407.01492)、[Aioli](https://arxiv.org/abs/2411.05735)、[DoGE](https://arxiv.org/abs/2310.15393) 和 [TANDEM](https://openreview.net/pdf?id=szBFUtBzWP)：

- **近乎匹配的场景（matched regime，$\Delta=0.07$）。** 这就是开头的七领域 SlimPajama 实验。训练与目标都对七个领域均匀加权，数据池足够大，不会因小领域耗尽而反复采样。最终平均困惑度在 19.66 到 20.17 之间，与 Uniform 的 19.71 相比，差异处于实验波动范围内。到了最后一个 checkpoint，早期领先已不再可靠。
- **内生 gap（endogenous gap，$\Delta=0.64$）。** 保持相同的均匀目标权重，但把数据池限制为 300M tokens，并保留原有领域比例。ArXiv、Wikipedia 等小领域很快耗尽，随后被反复采样。过拟合使这些领域的训练 loss 相对于留出目标 loss 更低。之所以叫“内生”，是因为 gap 来自有限数据上的训练过程。优化配比通过减少这些过拟合领域的曝光，改善了表现。
- **外生 gap（exogenous gap，$\Delta=1.26$）。** 把 CommonCrawl 留作目标，只在其余六个领域上训练。之所以叫“外生”，是因为候选训练池与目标本来就来自不同分布。虽然目标领域不在训练池中，C4 等相关数据仍提供了通过重新加权来部分纠正差距的可能。优化后，目标困惑度从 Uniform 的 49.19 降到约 40–41。

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/mixture-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*同一批配比方法，在近乎匹配的场景中，到最后一个 checkpoint 时收益很小；在存在可纠正 gap 的场景中，收益则保持到了测试过的最大预算。*

Selection 提供了另一组对照。我们固定模型、5B-token 的 [OpenWebMath](https://arxiv.org/abs/2310.06786) 候选池、60% 的保留比例，以及训练流程。比较方法包括全量数据持续预训练，以及 Random、[EL2N](https://arxiv.org/abs/2107.07075)、[InfoBatch](https://arxiv.org/abs/2303.04947)、[Token-Cleaning](https://proceedings.mlr.press/v267/pang25a.html)、[RHO-1](https://arxiv.org/abs/2404.07965) 和 [BLADE](https://arxiv.org/abs/2606.18650)。两种设置使用相同的方法，只改变选择和评价所用的目标。

- 目标同样来自 OpenWebMath 时，$\Delta=0.016$，没有一种筛选方法能稳定超过全量数据的持续预训练。
- 目标换成分布不同的 [MetaMath](https://arxiv.org/abs/2309.12284)–[MAmmoTH](https://arxiv.org/abs/2309.05653) 时，$\Delta=0.549$。利用目标信息的 RHO-1 和 BLADE 有了纠正可见差距的机会。其中，BLADE 只保留 60% 的候选 tokens，就把平均 benchmark 准确率从 17.21 提升到 31.31。

{% include figure.liquid loading="eager" path="assets/img/blog/data-curation/selection-gap-results.png" class="img-fluid rounded z-depth-1" zoomable=true %}

*在近乎匹配的场景中，selection 相比全量持续预训练收益很小；有可纠正的 gap 时，收益则明显增大。$\Delta$-test 可以在筛选之前区分这两种设置。*

## 换一种做事顺序

这些结果提示我们：在问“该用哪个数据优化方法”之前，应该先问“这个任务留出了怎样的改善空间”。$\Delta$-test 为这种判断提供了一个实用起点。

常见的流程从方法开始：

1. 设计一个数据价值评分。
2. 筛选数据，或者调整权重。
3. 训练模型。
4. 期待目标指标提升。

我更倾向于先看任务：

1. **定义目标。** 我们真正关心的能力，应由什么分布和指标来体现？
2. **运行基线，诊断任务。** 条件允许时，先用小型 proxy 模型估计 $\Delta$。
3. **检查 gap 是否可纠正。** 候选池是否包含所需内容？配比或筛选能否让模型更多地学到它们？
4. **明确要验证的收益，再优化数据。** 我们想节省计算、改善终点，还是同时做到二者？
5. **采用相应的验证方式。** 对加速，比较达到同等性能所需的 tokens、FLOPs 或实际耗时；对终点改善，增加预算，检验优势是否保持。

用更少计算达到相同能力，以及改善可达到的终点，都有价值。它们回答不同的问题，也需要不同的验证。无论是哪一种，改变数据的价值都取决于目标。这又引出一个问题：在说明希望模型学会什么之前，我们能否把训练数据称为“高质量”？

这是[下一篇：什么是好数据？（英文）]({{ '/blog/2026/what-makes-data-good/' | relative_url }})要讨论的问题。

{% include essay-footer.liquid citation_key='wang2026advancementzh' %}

</div>
