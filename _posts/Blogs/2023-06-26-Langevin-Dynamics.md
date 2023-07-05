---
layout: post
title: Bayesian Posterior Sampling (3) Langevin Dynamics
categories: Research
description: none
keywords: Bayesian, Langevin Dynamics, Fokker-Plank
mathjax: true
---

<style>
    .sidebar {
        float: right; /* Align the sidebar to the right */
        width: 300px; /* Set the width of the sidebar */
        font-family: sans-serif, monospace; /* Example font-family for a light font */
        margin-left: 30px; /* Add margin to the left of the sidebar */
    }
</style>

## Introduction
In the previous post, I presented some of the most classical MCMC method, _Metropolis-Hastings_ (MH) and its variants.
In this post I'd like to focus on another important MCMC method: Unadjusted Langevin Algorithm (ULA) in Bayesian statistics or Langevin Monte Carlo (LMC) in machine learning. LMC is a MCMC method which utilize Langevin dynamics for obtaining random samples from a probability distribution $\pi$ for which direct sampling is difficult (high-dimension and large number of data).

> Langevin dynamics provides an MCMC procedure to sample from a distribution $p(x)$ using only its score function $\nabla_x \log p(x)$. Specifically, it initializes the chain from an arbitrary prior distribution $x_0 \sim \pi(x)$, and then iterates the following:
> <div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    x_{i+1} \leftarrow x_i + \epsilon \nabla_x \log p(x) + \sqrt{2 \epsilon} z_i \label{Des_LD}\tag{1}
    \end{align*}$$ </div>
>
> ------ [_Generative Modeling by Estimating Gradients of the Data Distribution_](https://yang-song.net/blog/2021/score/) by Yang Song. Personal Blog, 2021.

## Langevin Diffusion and Fokker-Plank Equation
<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="overdamp">
            <sup>1</sup>Overdamped Langevin dynamics or Langevin dynamics without inertia: i.e. Langevin dynamics where no average acceleration (second derivativa of $X$ w.r.t. $t$) takes place.</p>
        <p style='margin-bottom: 5px;' id="GD">
            <sup>2</sup>This is closely related to the (stochastic) gradient descent algorithm.</p>
    </div>
</div>

#### Langevin Diffusion
The overdamped<a href="#overdamp"><sup>1</sup></a> Langevin Itô diffusion can be written as the following Stochastic Differential Equation (SDE):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    d{X_t}=\underbrace{-\nabla U(X_t)\mathrm{d}t}_{\text{drift term}} + \underbrace{\sqrt {2}{B_t}\mathrm{d}t}_{\text{diffusion term}},  \quad X_0\sim p_0 \label{LD}\tag{2}
    \end{align*}
$$
</div>
where $U(X)$ is the (time-dependent) potential energy function on $\mathbb{R}^d$ and $B_t$ is a d-dimensional standard Brownian Motion (or called the Wiener process). We assume that $U(X)$ is $L$-smooth (or $\nabla U(X)$ is $L$-Lipschitz): i.e. continuously
differentiable and $||\nabla U(x)-\nabla U(y)|| \leq L||x-y||, \exists L > 0$.

<!-- 
**Optimization**: find the minimum $min_{x\in \mathbb{R}^d} U(x)$ | **Sampling**: draw samples from the density $\pi(x)\propto e^{-U(x)}$
 -->

<!-- In our case, we have $\pi(x)=\frac{\exp(-\beta U(x))}{Z}$.  -->
In order to sample the diffusion paths, we can discrete (\ref{LD}) using the _Euler-Maruyama_ (EM) scheme as following<a href="#GD"><sup>2</sup></a> (similar to (\ref{Des_LD})):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    x_{i+1} = x_i - \gamma_{i+1} \nabla_x U(x_i) + \sqrt{2 \gamma_{i+1}} z_i \label{Des_LD2}\tag{3}
    \end{align*}
$$
</div>
where $z_i$ is i.i.d $\mathcal{N}(0,I_d)$ and $\gamma_{i}$ is the stepsize, either constant or decreasing to 0.

#### Fokker-Plank Equation
The Fokker-Planck (FP) equation, a form of partial differential equation (PDE), outlines how a probability distribution changes over time due to the influences of deterministic drift forces and stochastic fluctuations.

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="FP">
        <sup>3</sup>For derivation, see <a href="https://sites.me.ucsb.edu/~moehlis/moehlis_papers/appendix.pdf">this appendix</a>. You can also follow the derivation given by Kirill Neklyudov in a physics way in <a href="https://www.youtube.com/watch?v=3-KzIjoFJy4">this talk</a>.</p>
        <p style='margin-bottom: 10px;' id="FP2">
        <sup>4</sup>For FP equation without diffusion, there is a one slide derivation <a href="https://changliu00.github.io/static/d_mcmc.pdf">here</a> by Chang Liu.</p>
        <p style='margin-bottom: 10px;' id="Stationary">
        <sup>5</sup>Check out some quick derivations in <a href="https://stats.stackexchange.com/questions/414574/reconciling-langevin-mc-methods-as-one-step-hmc-versus-as-diffusion-or-brownian">this site</a>.</p>
    </div>
</div>

Let's denote the law of $X_t$ as $p_t$ and the FP equation for $p_t$ in (\ref{LD}) can be written as<a href="#FP"><sup>3</sup></a><sup>,</sup><a href="#FP2"><sup>4</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \partial_t p_t = \nabla\cdot(p_t\nabla U) + \Delta p_t \label{FP}\tag{4}
    \end{align*}
$$
</div>

To verify that $\pi$ is the unique stationary distribution for this FP equation (\ref{FP}) and hence the corresponding Langevin diffusion — which motivate us to use Langevin dynamics for sampling from $\pi$ — we can do any of the following<a href="#Stationary"><sup>5</sup></a>:
- substitute $p_t$ with $\pi$ to verify that it's a stationary distribution ($\partial_t \pi = 0$) 
- assume we already have a stationary distribution $p_\infty(x), s.t. \partial_t p_\infty=0$ and verify that $p_\infty(x) \propto \exp(-U(x))$. 
<!-- The readers are encouraged to try both as exercise. -->

#### Connection to Diffusion Models
<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="SDE">
        <sup>6</sup>Most of the case in ULA, the <em>drift term</em> is time independent and related to the target distribution, unlike the $f_t$ here in diffusion models.</p>
        <p style='margin-bottom: 10px;' id="FM">
        <sup>7</sup>You can find more in <a href="https://openreview.net/forum?id=PqvMRDCJT9t">this paper</a> on flow matching by Yaron Lipman <em>et al</em>.</p>
        <p style='margin-bottom: 10px;' id="PFODE">
        <sup>8</sup><a href="https://openreview.net/forum?id=PxTIG12RRHS">Score-Based Generative Modeling through Stochastic Differential Equations</a>  Y. Song, J. Sohl-Dickstein, D.P. Kingma, A. Kumar, S. Ermon, B. Poole. ICLR 2021.</p>
    </div>
</div>

For score-based models (or diffusion models) with forward SDE eq(\ref{LD}) in the form $d{X_t}={f_t(X)\mathrm{d}t} + {g_t{B_t}\mathrm{d}t}$<a href="#SDE"><sup>6</sup></a>, we can write the corresponding FP equation in the form of the _continuity equation_<a href="#FM"><sup>7</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \partial_t p_t = -\nabla\cdot(f_t p_t) + \frac{g_t^2}{2}\Delta p_t = -\nabla\cdot((f_t - \frac{g_t^2}{2} \nabla\log p_t) p_t) \label{FP2}\tag{5}
    \end{align*}
$$
</div>
where we define the vector field as $w_t = f_t - \frac{g_t^2}{2} \nabla\log p_t$, $g_t$ is somehow related to the temperature of the stationary distribution, and $p_0$ is the initial distribution (not necessarily Gaussian).

The corresponding Ordinary Differential Equations (ODE) of a particle moving along this vector field is the so-called <span style="color:#FFA000"><em>**probability flow ODE**</em></span><a href="#PFODE"><sup>8</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \mathrm{d} x = \bigg[f_t(x) - \frac{1}{2}g_t^2 \nabla_{x} \log p_t({x})\bigg] \mathrm{d}t \quad \Longleftrightarrow \quad \frac{\mathrm{d}x}{\mathrm{d}t} = v_t \label{prob_ode} \tag{6}
    \end{align*}
$$
</div>

Given $f_t$ and $g_t$ which define the forward diffusion process, we can travel backward in time with learned score function $s_\theta \approx \nabla \log p_t$ or the learned vector field $v_\theta \approx f_t - \frac{g_t^2}{2} \nabla\log p_t$.

## Langevin Monte Carle

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="ulaq">
        <sup>9</sup>Here is an <a href="https://fa.bianp.net/blog/2023/ulaq/">excellent blog</a> by Fabian Pedregosa on the convergence of ULA and why it's asymptotically biased.</p>
        <p style='margin-bottom: 10px;' id="lr">
        <sup>10</sup>Those moments I spent delicately tuning the learning rate in deep learning....</p>
    </div>
</div>

#### Unadjusted Langevin Algorithm

When we using eq(\ref{Des_LD2}) for sampling we are implementing the ULA.
Even though the stationary distribution of the Langevin diffusion is $\pi$, the stationary distribution of ULA is not due to the discretization<a href="#ulaq"><sup>9</sup></a>!
Note that when $\gamma_i$ is constant, the value of it controls the trade-off between the convergence speed and the accuracy<a href="#lr"><sup>10</sup></a>. 
ULA can be biased easily with larger step-size $\gamma_i$ and one could use a decreasing step-size or introduce the HM scheme to eliminate the bias.

<!-- The idea is that we have a great proposals and do not need decision rules anymore. -->

#### Metropolis-Adjusted Langevin Algorithm
The well-known MALA can be considered as MH with improved proposals or in this post ULA with an extra rejection step:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        g(x'|x) &= \mathcal{N}(x';x-\tau \nabla \tilde{\pi}(x),2\tau I) \\
        p_{\mathrm{accept}}(x') &= \min (1,\alpha) =\min \left(1, \frac{\tilde\pi\left(x^{\prime}\right) g\left(x | x^{\prime}\right)}{\tilde\pi(x) g\left(x^{\prime} | x\right)}\right)  \label{MALA}\tag{7}
    \end{align*}
$$
</div>
where the additional term $\tau \nabla \tilde{\pi}(x)$ drives the samples toward areas with high probability density, which makes the proposed samples more likely to be accepted. One significant advantage of MALA over Random Walk Metropolis (RWM) lies in the optimal step-size selection. In MALA, the asymptotic value for this parameter is larger compared to the RWM equivalent, leading to a decrease in the dependence observed between subsequent data points.


## Stochastic Gradient Langevin Dynamics
[Stochastic Gradient Langevin Dynamics](https://www.stats.ox.ac.uk/~teh/research/compstats/WelTeh2011a.pdf) (SGLD) is a method proposed by Max Welling and Yee Whye Teh where the traget density $\pi$ is the density of the **posterior distribution** $p(\theta|x)\propto {p(\theta)}\prod_{i=1}^{N}{p(x_i|\theta)}$. The authors initially observe the parallels between _stochastic gradient algorithms_ (that utilize a batch of samples in each iteration) and _Langevin dynamics_ (that employ all available samples in each iteration; one can consider ULA as gradient descent with some Gaussian noise added to the gradient at each iteration), and then propose a novel update rule that merges these two concepts with unbiased estimates of the gradient:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        \Delta\theta_{t}=\frac{\epsilon_{t}}{2}\left(\nabla\log p(\theta_{t})+\frac{N}{n}\sum_{i=1}^{n}\nabla\log p(x_{t i}|\theta_{t})\right)+\eta_{t}  \label{SGLD}\tag{8}
    \end{align*}
$$
</div>
where the step-sizes $\epsilon_{t}$ decrease towards zero, n is the batch size, and $\eta_{t} \sim \mathcal{N}(0,\epsilon_{t})$.

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="SGLD_ML">
        <sup>11</sup>SGLD is more related to machine learning.</p>
    </div>
</div>

Yes, SGLD is proposed to train a model given the dataset, and the idea is quite simple: We train the model using regular SGD, but add some Gaussian noise to each step. There are two sources of randomness: estimates of the gradient and Gaussian added noise to sample.

https://francisbach.com/rethinking-sgd-noise/

<!-- https://www.weideng.org/posts/CSGLD/ Dynamic Importance Sampling address the local trap issue. -->

<!-- ## Remarks -->
