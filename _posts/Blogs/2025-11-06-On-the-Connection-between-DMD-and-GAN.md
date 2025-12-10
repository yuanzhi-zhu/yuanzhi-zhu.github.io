---
layout: post
title: On the Connection between DMD and GAN for Diffusion Distillation
categories: Research
description: none
keywords: Diffusion Models, Diffusion Distillation, GAN
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


In this blog, I want to show a simple but intuitive connection between Variational Score Distillation (VSD/DMD) [1,2,3] and Diffusion-GAN [4] for diffusion distillation. 
This connection is noticed when I was trying to extend our recent workshop paper **Di-Bregman [5]**.

The final conclusion is not bonded to Di-Bregman, I just want to use Di-Bregman as an example to illustrate the connection.

---

## Preliminaries

In Di-Bregman, we derived a new distillation loss, whose loss gradient extend the DMD loss with an extra coefficient $h^{\prime\prime}(r_t(x_t))\,r_t(x_t)$:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\nabla_\theta \mathbb{D}_h(r_t\|1)
= -\mathbb{E}_{\epsilon,t}\Big[\,w(t)h^{\prime\prime}(r_t(x_t))\,r_t(x_t)\,\big(\nabla_{x_t}\log p_t({x_t})-\nabla_{x_t}\log q_{\theta,t}({x_t})\big)\,
\nabla_\theta G_\theta(\epsilon)\Big],
$$
</div>
where

- $r_t(x_t) = \frac{q_{\theta,t}(x_t)}{p_t(x_t)}$ is the density ratio between the student marginal $q_{\theta,t}$ and the teacher marginal $p_t$ at time $t$;
- $G_\theta(\epsilon)$ is the student generative model that maps noise $\epsilon$ to clean sample;
- $h(r)$ is a convex function defining the Bregman divergence $\mathbb{D}_h$;
- $w(t)$ is a time-dependent weight function;
- $\epsilon \sim \mathcal{N}(0,I)$, $t \sim \mathcal{U}(0,1)$, and $x_t = \alpha_t G_\theta(\epsilon) + \sigma_t z_t$ with $z_t \sim \mathcal{N}(0,I)$.

<div style="color:gray; font-size:0.85em; line-height:1.6;">
The DMD loss gradient derived from reverse KL divergence can be seen as a special case of the Di-Bregman when choosing $h(r)=r\log r$.
</div>

---

## From Scores to a Discriminator

One difference between Di-Bregman and DMD is that we need to estimate the density ratio $r_t(x_t)$ in the extra coefficiency $h^{\prime\prime}(r_t(x_t))\,r_t(x_t)$, usually achieved by training a discriminator $D(x_t,t)$ to distinguish samples from $p_t$ and $q_{\theta,t}$.

With the usual logistic convention (where $D_t(x_t)$ outputs the probability that $x_t$ comes from $p_t$), we have the following optimal discriminator $D_t^*(x_t)=\frac{p_t(x_t)}{p_t(x_t)+q_{\theta,t}(x_t)}$.

A natural question is: *can we rewrite the Di-Bregman loss gradient entirely in terms of the discriminator $D_t$, eliminating explicit score functions?*


### Step 1: Substitute the density-ratio gradient identity

We have the identity
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\nabla_{x_t} r_t({x_t}) = r_t({x_t})\,\big(\nabla_{x_t}\log q_{\theta,t}({x_t}) - \nabla_{x_t}\log p_t({x_t})\big)
$$
</div>

Substituting into the Di-Bregman gradient gives
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\begin{aligned}
\nabla_\theta \mathbb{D}_h(r_t\|1)
&= -\mathbb{E}_{\epsilon,t}\Big[w(t)\,h^{\prime\prime}(r_t)\,r_t\Big(-\frac{\nabla_{x_t} r_t}{r_t}\Big)\,\nabla_\theta G_\theta(\epsilon)\Big] \\
&= \mathbb{E}_{\epsilon,t}\Big[w(t)\,h^{\prime\prime}(r_t)\,\nabla_{x_t} r_t(x_t)\,\nabla_\theta G_\theta(\epsilon)\Big].
\end{aligned}
$$
</div>

### Step 2: Express $\nabla_{x_t} r_t$ and $r_{t}$ in terms of $D_t$

Since $r_t = \frac{q_{\theta,t}}{p_t} = \frac{1-D_t}{D_t}$, we have $\frac{dr_t}{dD_t} = -\frac{1}{D_t^2}$, hence
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\nabla_{x_t} r_t(x_t) = -\frac{1}{D_t(x_t)^2}\,\nabla_{x_t} D_t(x_t).
$$
</div>

Substitute this back:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\nabla_\theta \mathbb{D}_h(r_t\|1)
= -\mathbb{E}_{\epsilon,t}\Big[w(t)\,\frac{h^{\prime\prime}(\frac{1-D_t}{D_t})}{D_t^2}\,\nabla_{x_t} D_t(x_t)\,\nabla_\theta G_\theta(\epsilon)\Big].
$$
</div>

### Step 3: Convert $\nabla_{x_t} D_t$ to $\nabla_\theta D_t$
Using the chain rule, we have:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\boxed{
\nabla_\theta \mathbb{D}_h(r_t\|1)
= -\mathbb{E}_{\epsilon,t}\Big[
w'(t)\,\frac{h^{\prime\prime}(\frac{1-D_t}{D_t})}{D_t^2}\,
\nabla_\theta D_t(x_t,t)
\Big]
}
$$
</div>

> The conversion from $\nabla_{x_t}D_t$ to $\nabla_\theta D_t$ introduces the factor $\alpha_t$, which is absorted into $w'(t)$.

The boxed equation shows that the Di-Bregman loss gradient can be computed by backpropagating through the discriminator $D_t$ only, without explicitly estimating the score functions. 
This suggests that **Di-Bregman / VSD and Diffusion-GAN are closely connected**.

<!-- In experiments, we can show that score difference term is more stable than the direct discriminator gradient. -->

---

## Verify with Special Case: reverse KL Divergence (DMD)

When $h(r)=r\log r$, we have $h^{\prime\prime}(r)=\frac{1}{r}$, hence the coefficiency in the boxed equation becomes $\frac{h^{\prime\prime}(\frac{1-D_t}{D_t})}{D_t^2} = \frac{1}{D_t(1-D_t)}$, and the loss gradient becomes
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\nabla_\theta \mathbb{D}_{\mathrm{KL}}(q_{\theta,t}\|p_t)
= -\mathbb{E}_{\epsilon,t}\Big[ w'(t)\,\frac{1}{D_t(1-D_t)}\,\nabla_\theta D_t(x_t,t) \Big].
$$
</div>

This resonates with the GAN generate loss for KL divergence $\mathrm{KL}(q_\theta||p)$:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\mathcal{L}_G = -\mathbb{E}_{\epsilon}\Big[\log \frac{D(G_\theta(\epsilon))}{1-D(G_\theta(\epsilon))}\Big].
$$
</div>

<div style="color:gray; font-size:0.85em; line-height:1.6;">
This special case was also noticed in Diff-instruct [3] Corollary 3.5 (From KL perspective to derive both losses).
</div>

---

> Given that both DMD and Di-Bregman are equivalent to diffusion-GAN training, and with the belief that scalable pre-training requires algorithms that directly optimize the data likelihood or an associated ELBO, I am skeptical that these approaches can serve as general-purpose pre-training methods.
> For example, it seems unlikely that we could pre-train a next-frame prediction video generative model using methods such as self-forcing.

## References

<div style="color:gray; font-size:0.85em; line-height:1.6;">

<p>
  <b>[1]</b> <i>Prolificdreamer: High-fidelity and diverse text-to-3d generation with variational score distillation.</i><br>
  Zhengyi Wang, Cheng Lu, Yikai Wang, Fan Bao, Chongxuan Li, Hang Su, and Jun Zhu. Advances in neural information processing systems 36 (2023).
</p>

<p>
  <b>[2]</b> <i>One-step diffusion with distribution matching distillation.</i><br>
  Tianwei Yin, Michaël Gharbi, Richard Zhang, Eli Shechtman, Fredo Durand, William T. Freeman, and Taesung Park. In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, pp. 6613-6623. 2024.
</p>

<p>
  <b>[3]</b> <i>Diff-instruct: A universal approach for transferring knowledge from pre-trained diffusion models.</i><br>
  Weijian Luo, Tianyang Hu, Shifeng Zhang, Jiacheng Sun, Zhenguo Li, and Zhihua Zhang. Advances in Neural Information Processing Systems 36 (2023).
</p>

<p>
  <b>[4]</b> <i>Diffusion-gan: Training gans with diffusion.</i><br>
  Zhendong Wang, Huangjie Zheng, Pengcheng He, Weizhu Chen, and Mingyuan Zhou. arXiv preprint arXiv:2206.02262 (2022).
</p>

<p>
  <b>[5]</b> <i>One-step Diffusion Models with Bregman Density Ratio Matching.</i><br>
  Yuanzhi Zhu, Eleftherios Tsonis, Lucas Degeorge, Vicky Kalogeiton. arXiv preprint, arXiv:2510.16983, 2025.
</p>

</div>
