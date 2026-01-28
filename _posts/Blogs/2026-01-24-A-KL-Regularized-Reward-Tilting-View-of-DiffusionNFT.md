---
layout: post
title: A KL-Regularized Reward-Tilting View of DiffusionNFT
date: 2026-01-24
categories: Research
description: none
keywords: Diffusion Models, Diffusion RL, DiffusionNFT
mathjax: true
authors:
  - "Yuanzhi Zhu"
---

<style>
    .sidebar {
        float: right; /* Align the sidebar to the right */
        width: 300px; /* Set the width of the sidebar */
        font-family: sans-serif, monospace; /* Example font-family for a light font */
        margin-left: 30px; /* Add margin to the left of the sidebar */
    }
</style>

<!-- This note rewrites the DiffusionNFT objective in distributional form. The key point is that, while the paper is phrased as a velocity-field (or score/flow) update with positive/negative splits, the induced density-level optimum is an explicit KL-regularized exponential tilt of a reference distribution—where the “reward” is an optimality posterior at the noisy state.

--- -->

## Offline DiffusionNFT as KL-Regularized Reward Tilting

### Setup (From DiffusionNFT)

Introduce a binary optimality variable $o\in\{0,1\}$ and a prompt/context $c$.
DiffusionNFT defines the reward as an optimality probability

$$
r(x_0,c)\;:=\;p(o=1\mid x_0,c)\in[0,1].
$$

Given a reference (old) model $\pi_{\mathrm{old}}(x_0\mid c)$, define the positive (optimality-conditioned) distribution

$$
\pi_{+}(x_0\mid c)
:=\pi_{\mathrm{old}}(x_0\mid o=1,c)
=\frac{p(o=1\mid x_0,c)}{p_{\mathrm{old}}(o=1\mid c)}\,\pi_{\mathrm{old}}(x_0\mid c)=
\frac{r(x_0,c)}{\mathbb{E}_{x_0\sim \pi_{\mathrm{old}}(\cdot\mid c)}[r(x_0,c)]}\;
\pi_{\mathrm{old}}(x_0\mid c),
$$

and the corresponding forward-noised marginals at diffusion time $t$ under a fixed kernel $\pi_{t\mid 0}(x_t\mid x_0)$:

$$
p^{\mathrm{old}}_t(x_t\mid c)=\int \pi_{t\mid 0}(x_t\mid x_0)\,\pi_{\mathrm{old}}(x_0\mid c)\,dx_0,
\qquad
p^{+}_t(x_t\mid c)=\int \pi_{t\mid 0}(x_t\mid x_0)\,\pi_{+}(x_0\mid c)\,dx_0.
$$

DiffusionNFT's training objective yields an optimal velocity field of the form

$$
v^*(x_t,c,t)=v^{\mathrm{old}}(x_t,c,t)+\frac{2}{\beta}\,\Delta(x_t,c,t),
$$

with guidance direction $\Delta(x_t,c,t)=\alpha(x_t,c)\bigl(v^{+}(x_t,c,t)-v^{\mathrm{old}}(x_t,c,t)\bigr)$.

And the mixture coefficient is defined as

$$
\alpha(x_t,c):=\frac{p^{+}_t(x_t\mid c)}{p^{\mathrm{old}}_t(x_t\mid c)}\;\mathbb{E}_{x_0\sim \pi_{\mathrm{old}}(\cdot\mid c)}[r(x_0,c)].
$$

#### Derivation of $\alpha(x_t,c)=p(o=1\mid x_t,c)$

Apply the forward diffusion to the positive distribution:

$$
\frac{p^{+}_t(x_t\mid c)}{p^{\mathrm{old}}_t(x_t\mid c)}
=
\frac{p_{\mathrm{old}}(o=1\mid x_t,c)}{p_{\mathrm{old}}(o=1\mid c)}.
$$

where we use the identity

$$
p_{\mathrm{old}}(o=1\mid c)=\mathbb{E}_{x_0\sim \pi_{\mathrm{old}}(\cdot\mid c)}[r(x_0,c)],
$$

we obtain

$$
\boxed{
\alpha(x_t,c)=p_{\mathrm{old}}(o=1\mid x_t,c).
}
$$

Note that $\alpha(x_t,c)=\mathbb{E}_{x_0\sim \pi _{\mathrm{old}}(\cdot \mid c)}[r(x_0,c)\mid x_t]$. This shows that the mixture coefficient equals the optimality posterior distribution at the noisy state $x_t$ under the old model.


#### Remark

This equality is a direct consequence of the definition of $\pi_{+}$ as an optimality-conditioned distribution and Bayes' rule under the fixed forward noising kernel. It may be omitted or only implicit in the original DiffusionNFT paper.


### DiffusionNFT optimal distribution at each step

In order to compute the optimal distribution, we need to simplify the residual term $\Delta(x_t,c,t)=\alpha(x_t,c)\bigl(v^{+}(x_t,c,t)-v^{\mathrm{old}}(x_t,c,t)\bigr)$.

<!-- Under the standard Gaussian perturbation family (fixed forward noising), velocity differences reduce to score differences:

$$
v^{A}(x_t,c,t)-v^{B}(x_t,c,t)=\kappa(t)\Big(\nabla_{x_t}\log p^{A}_t(x_t\mid c)-\nabla_{x_t}\log p^{B}_t(x_t\mid c)\Big),
$$ -->

<!-- where $\kappa(t)$ depends only on the noise schedule. -->

Using the Bayes relation for the positive marginal

$$
p^{+}_t(x_t\mid c)=p^{\mathrm{old}}_t(x_t\mid c)\,\frac{p(o=1\mid x_t,c)}{p(o=1\mid c)},
$$

and i) utilizing the relation between velocity fields and score functions under fixed Gaussian noising, and ii) applying log and gradient, one has

$$
\Delta(x_t,c,t) \propto \frac{2}{\beta}\,p(o=1\mid x_t,c)\,\nabla_{x_t}\log p(o=1\mid x_t,c).
$$

<!-- $$
\nabla_{x_t}\log p^{*}_t(x_t\mid c)
=
\nabla_{x_t}\log p^{\mathrm{old}}_t(x_t\mid c)
+\frac{2}{\beta}\,p(o=1\mid x_t,c)\,\nabla_{x_t}\log p(o=1\mid x_t,c).
$$ -->

<!-- one obtains the score-level characterization of the optimum (since $p(o=1\mid c)$ is independent of $x_t$): -->

#### Closed-form optimal distribution (density-level)

Since $\alpha(x_t,c)=p(o=1\mid x_t,c)$ and $\alpha\nabla\log\alpha=\nabla\alpha$, we have:

$$
\nabla_{x_t}\log\frac{p^{*}_t(x_t\mid c)}{p^{\mathrm{old}}_t(x_t\mid c)}
=\frac{2}{\beta}\,\nabla_{x_t}\alpha(x_t,c).
$$

This implies the explicit density:

$$
p^{*}_t(x_t\mid c)
=
\frac{1}{Z_t(c)}\;p^{\mathrm{old}}_t(x_t\mid c)\;
\exp\!\Big(\frac{2}{\beta}\,p(o=1\mid x_t,c)\Big),
\qquad
Z_t(c)=\int p^{\mathrm{old}}_t(x_t\mid c)\exp\!\Big(\frac{2}{\beta}p(o=1\mid x_t,c)\Big)\,dx_t.
$$

At $t=0$ this reduces to $p^{*}(x_0\mid c)\propto p^{\mathrm{old}}(x_0\mid c)\exp(\frac{2}{\beta}r(x_0,c))$.

<!-- #### Comparison to standard KL-regularized reward fine-tuning

A common formulation of reward fine-tuning with KL regularization has the exponential-tilt optimum

$$
p^{\mathrm{tilt}}_\lambda(x)
=
\frac{1}{Z(\lambda)}\,p^{\mathrm{ref}}(x)\exp\big(\lambda\,R(x)\big),
$$

equivalently the solution to $\max_p \mathbb{E}_{p}[R(x)]-\frac{1}{\lambda}\mathrm{KL}(p\|p^{\mathrm{ref}})$.
Comparing with the closed form above, DiffusionNFT's optimum is exactly a reward-tilted distribution with

$$
p^{\mathrm{ref}}_t(\cdot\mid c)=p^{\mathrm{old}}_t(\cdot\mid c),\qquad
R_t(x_t,c)=p(o=1\mid x_t,c),\qquad
\lambda=\frac{2}{\beta}.
$$ -->

Thus, DiffusionNFT with fixed $p^{\mathrm{old}}=p^{\mathrm{ref}}$ can be viewed as learning a KL-regularized exponential tilt of the reference model, where the “reward” is the optimality posterior at the noisy state.

<!-- #### Relation to control-as-inference parameterizations

In control-as-inference, one often defines a log-potential reward $\tilde R$ via

$$
p(o=1\mid x)\propto \exp(\tilde R(x)/\tau)
$$

(possibly with offsets/temperature to ensure proper probabilities). DiffusionNFT instead takes $r(x)=p(o=1\mid x)$ directly, so the mapping between conventions is $\tilde R(x)=\tau\log p(o=1\mid x)$ (up to an additive constant). -->


<!-- #### Conclusion

Although DiffusionNFT derives an optimal velocity-field update $v^*=v^{\mathrm{old}}+\frac{2}{\beta}\Delta$ and interprets policy improvement via positive/negative splits, the corresponding density-level optimum can be written explicitly as the reward-tilted distribution above. This KL-regularized exponential-tilt characterization is implicit in the paper's theorems but is not typically stated in closed form as an explicit $p^*$. -->

---

## Online DiffusionNFT Leads to Reward Hacking via Accumulated Tilting

<!-- The previous subsection characterizes the per-step (single-stage) population optimum when the reference $v^{\mathrm{old}}$ (equivalently $p^{\mathrm{old}}_t$) is held fixed. -->

In practice, DiffusionNFT can be run online: after fitting $v_\theta$ for one epoch, the old model is updated (e.g., by copying weights (hard) or by EMA (soft)) and the next epoch is trained against this updated old model. This induces a recursion over the initial reference distributions.

### Idealized recursion with exact per-epoch optima

Let $p^{(k)}_t(x_t\mid c)$ denote the old marginal at epoch $k$ (the distribution induced by the current old velocity field). Treat the reward/optimality posterior $\alpha_t(x_t,c)=p(o=1\mid x_t,c)$ as fixed across epoches.

From the closed-form optimum, the population-optimal update at epoch $k$ is

$$
p^{(k+1)}_t(x_t\mid c)
=
\frac{1}{Z^{(k)}_t(c)}\;p^{(k)}_t(x_t\mid c)\;
\exp\!\Big(\lambda\,\alpha_t(x_t,c)\Big),
\qquad
\lambda=\frac{2}{\beta}.
$$

Unrolling the recursion yields

$$
p^{(K)}_t(x_t\mid c)
\propto
p^{(0)}_t(x_t\mid c)\;
\exp\!\Big(\lambda K\,\alpha_t(x_t,c)\Big).
$$

<!-- Hence, under exact per-stage optimization and a fixed $\alpha_t$, online training compounds the reward tilt: the effective inverse temperature grows linearly with the number of stages. -->

<!-- ### Concentration in the large-stage limit -->

As $K\to\infty$, the previous expression concentrates mass on the set of maximizers of the reward.


<!-- Formally, let $\alpha_{\max}(c)=\sup_{x_t}\alpha_t(x_t,c)$ and $S(c)=\{x_t:\alpha_t(x_t,c)=\alpha_{\max}(c)\}$.
Under mild regularity assumptions (e.g., $p^{(0)}_t(\cdot\mid c)$ has nonzero mass near $S(c)$), the family $p^{(K)}_t(\cdot\mid c)$ converges to a distribution supported on $S(c)$, with relative weights inherited from $p^{(0)}_t(\cdot\mid c)$ restricted to $S(c)$. -->

### Remarks on EMA references

If the reference is updated via EMA in parameter space, the induced distribution recursion is not exactly the simple multiplicative update above. Nevertheless, EMA typically acts as a trust-region mechanism that interpolates between keeping $p^{(k)}_t$ fixed and fully replacing it by the latest student, effectively reducing the rate at which the tilt coefficient grows with $k$. The qualitative conclusion remains: in the absence of an anchoring term to the initial model, repeated online improvement tends to accumulate reward tilt and can become increasingly peaked around high-reward regions and thus prone to reward hacking.

---

## Online DiffusionNFT with an Additional KL to the Initial Reference

To prevent unbounded drift from the original model, one can augment the per-epoch objective with an additional regularizer that penalizes deviation from the initial reference distribution $p^{(0)}_t(\cdot\mid c)=p^{\mathrm{ref}}_t(\cdot\mid c)$.

At the distribution level, consider the KL-regularized problem at epoch $k$:

$$
p^{(k+1)}_t
=
\arg\max_{p_t}\;
\mathbb{E}_{x_t\sim p_t}\!\big[\alpha_t(x_t,c)\big]
-\frac{1}{\eta_1}\mathrm{KL}\!\big(p_t\|p^{(k)}_t\big)
-\frac{1}{\eta_0}\mathrm{KL}\!\big(p_t\|p^{(0)}_t\big),
\qquad \eta_0,\eta_1>0.
$$

Here $\eta_1$ controls the trust region to the current reference and $\eta_0$ controls anchoring to the initial reference.

The unique optimizer has the closed form

$$
p^{(k+1)}_t(x_t\mid c)
=
\frac{1}{\widetilde Z^{(k)}_t(c)}\;
\Big(p^{(k)}_t(x_t\mid c)\Big)^{w}\,
\Big(p^{(0)}_t(x_t\mid c)\Big)^{1-w}\,
\exp\!\Big(\lambda_{\mathrm{eff}}\,\alpha_t(x_t,c)\Big),
$$

with weights and effective tilt coefficient

$$
w=\frac{\eta_0}{\eta_0+\eta_1}\in(0,1),
\qquad
\lambda_{\mathrm{eff}}=\frac{\eta_0\eta_1}{\eta_0+\eta_1}.
$$

Thus, the per-epoch solution is an exponential tilt of a geometric mixture of the current and initial references.

<!-- ### Recursive application and limiting distribution

Assume the iterates maintain the exponential-tilt form

$$
p^{(k)}_t(x_t\mid c)
=
\frac{1}{Z^{(k)}_t(c)}\;p^{(0)}_t(x_t\mid c)\exp\!\big(B_k\,\alpha_t(x_t,c)\big),
$$

which holds when initialized at $p^{(0)}_t$ and applying the two-KL update with fixed $\alpha_t$.

Substituting into the closed form yields the scalar recursion

$$
B_{k+1}=w\,B_k+\lambda_{\mathrm{eff}},
\qquad B_0=0.
$$

Solving gives

$$
B_k=\frac{\lambda_{\mathrm{eff}}}{1-w}\big(1-w^k\big)=\eta_0\big(1-w^k\big),
\qquad\Rightarrow\qquad
\lim_{k\to\infty}B_k=\eta_0.
$$

Therefore, the online procedure converges to the finite tilt -->

It is straightforward to see that the iterates converge to a limiting distribution:

$$
\boxed{
p^{(\infty)}_t(x_t\mid c)
=
\frac{1}{Z^{(\infty)}_t(c)}\;p^{(0)}_t(x_t\mid c)\exp\!\big(\eta_0\,\alpha_t(x_t,c)\big).
}
$$

Notably, the limiting distribution depends only on the anchoring strength $\eta_0$; the trust-region parameter $\eta_1$ affects only the dynamics (convergence rate and stability) through $w$.

<!-- ### Interpretation

Adding $\mathrm{KL}(p_t\|p^{(0)}_t)$ transforms online DiffusionNFT from an accumulating reward-tilt procedure into one with a well-defined stationary solution. This provides a principled mechanism to cap distributional drift from the original reference while still allowing progress toward higher optimality posterior regions. -->

<!-- --- -->

---

> In practice, the finetuned model in DiffusionNFT is initialized from a pre-trained diffusion model without CFG. Moreover, the initial reference model is also the pre-trained diffusion model without CFG. 
> Thus, with large inital KL strength, the online DiffusionNFT procedure effectively regularizes the finetuned model toward the pre-trained diffusion model without CFG and the learned model can only generate blurry samples; with small initial KL strength used in the experiments in the paper, the finetuned model generates samples with high reward but low diversity and pure color background, which suggests severe reward hacking.
> Based on the analysis above, we suggest adding an medium-strength initial KL regularization and inference the pre-trained model with CFG as the initial reference to mitigate reward hacking in online DiffusionNFT.

## References

<div style="color:gray; font-size:0.85em; line-height:1.6;">

<p>
  <b>[1]</b> <i>DiffusionNFT: Online Diffusion Reinforcement with Forward Process.</i><br>
  Kaiwen Zheng, Huayu Chen, Haotian Ye, Haoxiang Wang, Qinsheng Zhang, Kai Jiang, Hang Su, Stefano Ermon, Jun Zhu, and Ming-Yu Liu. arXiv preprint arXiv:2509.16117 (2025).
</p>

<p>
  <b>[2]</b> <i>Test-time Alignment of Diffusion Models without Reward Over-optimization.</i><br>
  Sunwoo Kim, Minkyu Kim, and Dongmin Park. The Thirteenth International Conference on Learning Representations (ICLR 2025).
</p>

<p>
  <b>[3]</b> <i>Feedback Efficient Online Fine-Tuning of Diffusion Models.</i><br>
  Masatoshi Uehara, Yulai Zhao, Kevin Black, Ehsan Hajiramezanali, Gabriele Scalia, Nathaniel Lee Diamant, Alex M. Tseng, Sergey Levine, and Tommaso Biancalani. Proceedings of the 41st International Conference on Machine Learning (ICML 2024).
</p>

</div>
