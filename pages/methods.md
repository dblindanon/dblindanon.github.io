---
layout: default
title: Methods
permalink: /methods
---

## R.1: Methods

We briefly review the main method proposed in the paper, *Robust MoE*, and then describe an additional variant, *Robust Filtered*. The goal of both methods is the same: to make soft-routed mixture-of-experts models less fragile under distribution shift by focusing training on the examples where routing-induced errors are most consequential.

### Robust MoE (proposed method)

The main method proposed in the paper is Robust MoE. In a soft-routed MoE, mixture-level calibration is especially fragile in *routing overlap regions* where multiple experts receive substantial weight and give different predictions. These regions are often uncommon in the training distribution but can become much more prevalent under distribution shift. When that happens, the aggregate predictor can become badly miscalibrated.

To address this, Robust MoE replaces standard average-risk minimization with a distributionally robust objective

$$\min_{(r,\{f_k\})} \sup_{w \in \mathcal{W}_\Gamma} \mathbb{E}_{P_\text{train}} \left[ w(X) \, \ell(f(X), Y) \right], \quad f(x) = \sum_{k=1}^K r_k(x) \, f_k(x),$$

where \\(\mathcal{W}_\Gamma\\) is a family of bounded reweightings of the training distribution. The inner supremum finds the worst-case reweighting of the training distribution, forcing the model to perform well even on adversarially upweighted subpopulations.

As discussed in the paper, this objective has a CVaR-like interpretation: rather than treating all training examples equally, it places more emphasis on the highest-loss part of the distribution. In practice, this means the model is encouraged to improve on brittle regions like routing overlap regions.

### Robust Filtered (additional variant)

In addition to the paper's main method, we also report results for a new variant, *Robust Filtered*. This is not part of the submitted paper, but we include it here as a useful extension with the same underlying idea.

Robust Filtered restricts the CVaR emphasis to the samples where MoE routing actually matters. Some samples are universally hard, where all experts struggle, so emphasizing these may increase optimization pressure without specifically addressing the calibration problems that come from routing fragility. To avoid this, Robust Filtered applies the robust tail emphasis only to *routing-relevant* examples. We identify an example as routing-relevant when at least one of the following holds:

- the mixture prediction is worse than its best expert, suggesting that routing is actively hurting performance, or
- the experts disagree substantially, indicating that the choice of routing weights has a meaningful effect on the final prediction. Specifically, we compute \\(\sum_{k} r_k(x) \\| f_k(x) - f(x) \\|^2\\), which measures how much the individual expert predictions deviate from the mixture output, weighted by each expert's routing probability. When this exceeds a threshold (default 0.01), routing is materially affecting the prediction.

The total loss combines a standard ERM term over all samples with a CVaR term applied only to the routing-relevant subset: \\(\mathcal{L} = \mathcal{L}\_\text{ERM} + \mathcal{L}\_\text{CVaR}^\text{filtered}\\). Samples that are not routing-relevant contribute only through the ERM gradient. We hope that this filtering makes the robustness objective more targeted: it can retain the calibration benefits of Robust MoE on routing-sensitive subpopulations, while reducing the accuracy cost that may come from aggressively upweighting universally difficult examples.