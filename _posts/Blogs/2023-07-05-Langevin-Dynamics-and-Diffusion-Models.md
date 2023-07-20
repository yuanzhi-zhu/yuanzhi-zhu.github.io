---
layout: post
title: Bayesian Posterior Sampling (3) Langevin Dynamics and Diffusion Models
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

## Langevin Dynamics Sampling 
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
In order to sample the diffusion paths, we can discrete eq(\ref{LD}) using the _Euler-Maruyama_ (EM) scheme as following<a href="#GD"><sup>2</sup></a> (similar to eq(\ref{Des_LD})):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    x_{i+1} = x_i - \gamma_{i+1} \nabla_x U(x_i) + \sqrt{2 \gamma_{i+1}} z_i \label{Des_LD2}\tag{3}
    \end{align*}
$$
</div>
where $z_i$ is i.i.d $\mathcal{N}(0,I_d)$ and $\gamma_{i}$ is the stepsize, either constant or decreasing to 0.


<span style="color:gray">
The subject of mixing time in MCMC algorithms is quite complex and necessitates substantial analysis skills. Hence, we won't delve into it in this blog post.
</span>

#### Fokker-Plank Equation
The Fokker-Planck (FP) equation, a form of partial differential equation (PDE), outlines how a probability distribution changes over time due to the influences of deterministic drift forces and stochastic fluctuations.

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="FP">
        <sup>3</sup>For derivation, see <a href="https://sites.me.ucsb.edu/~moehlis/moehlis_papers/appendix.pdf">this appendix</a>. You can also follow the derivation given by Kirill Neklyudov in a physics way in <a href="https://www.youtube.com/watch?v=3-KzIjoFJy4">this talk</a>.</p>
        <p style='margin-bottom: 10px;' id="FP2">
        <sup>4</sup>For FP equation without diffusion, there is a one slide derivation <a href="https://changliu00.github.io/static/d_mcmc.pdf">here</a> by Chang Liu.</p>
        <p style='margin-bottom: 10px;' id="Stationary">
        <sup>5</sup>Check out some quick derivations on <a href="https://stats.stackexchange.com/questions/414574/reconciling-langevin-mc-methods-as-one-step-hmc-versus-as-diffusion-or-brownian">this site</a>.</p>
    </div>
</div>

Let's denote the law of $X_t$ as $p_t$ and the FP equation for $p_t$ in eq(\ref{LD}) can be written as<a href="#FP"><sup>3</sup></a><sup>,</sup><a href="#FP2"><sup>4</sup></a>:
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

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="ulaq">
        <sup>6</sup>Here is an <a href="https://fa.bianp.net/blog/2023/ulaq/">excellent blog</a> by Fabian Pedregosa on the convergence of ULA and why it's asymptotically biased.</p>
        <p style='margin-bottom: 10px;' id="lr">
        <sup>7</sup>Those moments I spent delicately tuning the learning rate in deep learning....</p>
    </div>
</div>

#### Unadjusted Langevin Algorithm

When we using eq(\ref{Des_LD2}) for sampling we are implementing the ULA.
Even though the stationary distribution of the Langevin diffusion is $\pi$, the stationary distribution of ULA is not, due to the discretization<a href="#ulaq"><sup>6</sup></a>!
Note that when $\gamma_i$ is constant, the value of it controls the trade-off between the convergence speed and the accuracy<a href="#lr"><sup>7</sup></a>. 
ULA can be biased easily with larger step-size $\gamma_i$ and one could use a decreasing step-size or introduce the HM scheme to eliminate the bias.

<!-- The idea is that we have a great proposals and do not need decision rules anymore. -->

#### Metropolis-Adjusted Langevin Algorithm
The well-known MALA can be considered as MH with improved proposals or in this post ULA with an extra rejection step:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        g(x'|x) &= \mathcal{N}(x';x-\tau \nabla \tilde{\pi}(x),2\tau I) \\
        p_{\mathrm{accept}}(x') &= \min (1,\alpha) =\min \left(1, \frac{\tilde\pi\left(x^{\prime}\right) g\left(x | x^{\prime}\right)}{\tilde\pi(x) g\left(x^{\prime} | x\right)}\right)  \label{MALA}\tag{5}
    \end{align*}
$$
</div>
where the additional term $\tau \nabla \tilde{\pi}(x)$ drives the samples toward areas with high probability density, which makes the proposed samples more likely to be accepted. One significant advantage of MALA over Random Walk Metropolis (RWM) lies in the optimal step-size selection. In MALA, the asymptotic value for this parameter is larger compared to the RWM equivalent, leading to a decrease in the dependence observed between subsequent data points.

<!-- 
<div style="overflow-x: auto; white-space: nowrap; margin-top: 0px;">
<center>
<object data="https://chi-feng.github.io/mcmc-demo/app.html?algorithm=RandomWalkMH&target=donut" type="text/html" width="600" height="400">
</object>
</center>
<p align="center">
<em>Figure 1: Visual illustration of random walk Metropolis-Hastings in 2D space<a href="#HD4"><sup>9</sup></a>.</em>
</p>
</div> -->

#### Stochastic Gradient Langevin Dynamics
[Stochastic Gradient Langevin Dynamics](https://www.stats.ox.ac.uk/~teh/research/compstats/WelTeh2011a.pdf) (SGLD) is a method proposed by Max Welling and Yee Whye Teh where the traget density $\pi$ is the density of the **posterior distribution** $p(\theta|x)\propto {p(\theta)}\prod_{i=1}^{N}{p(x_i|\theta)}$. The authors initially observe the parallels between _stochastic gradient algorithms_ (that utilize a batch of samples in each iteration) and _Langevin dynamics_ (that employ all available samples in each iteration; one can consider ULA as gradient descent with some Gaussian noise added to the gradient at each iteration), and then propose a novel update rule that merges these two concepts with unbiased estimates of the gradient:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        \Delta\theta_{t}=\frac{\epsilon_{t}}{2}\left(\nabla\log p(\theta_{t})+\frac{N}{n}\sum_{i=1}^{n}\nabla\log p(x_{t i}|\theta_{t})\right)+\eta_{t}  \label{SGLD}\tag{6}
    \end{align*}
$$
</div>
where the step-sizes $\epsilon_{t}$ decrease towards zero, n is the batch size, and $\eta_{t} \sim \mathcal{N}(0,\epsilon_{t})$.

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="SGLD_ML">
        <sup>8</sup>SGLD is primarily associated with the field of machine learning. There are numerous informative resources available on this topic, such as <a href="https://francisbach.com/rethinking-sgd-noise/">this blog</a>.</p>
    </div>
</div>

Yes, SGLD is proposed to train a model given the dataset<a href="#SGLD_ML"><sup>8</sup></a>, and the idea is quite simple: We train the model using regular SGD, but add some Gaussian noise to each step. There are two sources of randomness: estimates of the gradient and Gaussian added noise to sample.

<!-- https://www.weideng.org/posts/CSGLD/ Dynamic Importance Sampling address the local trap issue. -->

## From Langevin Dynamics to Diffusion Models

#### Pitfalls with Unadjusted Langevin Dynamics
<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="sm">
        <sup>9</sup><a href="https://www.iro.umontreal.ca/~vincentp/Publications/smdae_techreport.pdf">A Connection between Score Matching and Denoising Autoencoders</a> P. Vincent. Neural computation, Vol 23(7), pp. 1661--1674. MIT Press. 2011.
        </p>
        <p style='margin-bottom: 10px;' id="pitfall">
        <sup>10</sup><a href="https://proceedings.neurips.cc/paper_files/paper/2019/file/3001ef257407d5a371a96dcd947c7d93-Paper.pdf">Generative Modeling by Estimating Gradients of the Data Distribution</a> Y. Song, S. Ermon. NeuIPS 2019.
        </p>
        <p style='margin-bottom: 10px;' id="smdsm">
        <sup>11</sup>In <a href="https://kexue.fm/archives/9509">this blog</a> by Jianlin Su you can find some discussion (in Chinese) about the relationship between denoising score matching and score matching.
        </p>
        <p style='margin-bottom: 10px;' id="stlmc">
        <sup>12</sup>In <a href="http://www.offconvex.org/2021/03/01/beyondlogconcave2/">this blog</a> the authors introduced a very similar technique called <em>Simulated Tempering Langevin Monte Carlo</em>.
        </p>
    </div>
</div>

<!-- However, with only access to (learnt) $\nabla U_\theta$ we are not able to generate samples from the target distribution in high-dimensional space due to several practical pitfalls<a href="#pitfall"><sup>9</sup></a>.  -->

Assume now we have access to oracle $s_\theta \approx \nabla \log p_\infty = -\nabla U$ through score matching<a href="#sm"><sup>9</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \theta^{\star} = & \arg \min \frac{1}{2}\mathbb{E}_{p_{\mathrm{data}}(x)} \left[||s_\theta (x)-\nabla_{x} \log p_{\mathrm{data}}(x)||_{2}^{2} \right] \\ 
    = & \arg \min \mathbb{E}_{p_{\mathrm{data}}(x)}\left[\mathrm{tr}(\nabla_{x}{s}_\theta(x))+\frac{1}{2}||{s}_\theta(x)||_{2}^{2}\right], \label{sm}\tag{7}
    \end{align*}
$$
</div>
are we safe to use Langevin Algorithm (\ref{Des_LD2}) for image generation in high dimension? No, there are pitfalls<a href="#pitfall"><sup>10</sup></a>! 
- The [manifold hypothesis](https://en.wikipedia.org/wiki/Manifold_hypothesis): the score function $s_\theta$ is ill-defined when $x$ is confined to a low dimensional manifold in a high-dimensional space; and the score matching objective in eq(\ref{sm}) will provides a inconsistent score estimator when the data reside on a low-dimensional manifold.
- The scarcity of data in low density regions can cause difficulties for both _score estimation with score
matching_ and _MCMC sampling with Langevin dynamics_ (suffers from slow mixing time due to separate regions of data manifold).

#### Annealed Langevin Dynamics
To overcome these issues, Yang Song _et al_ proposed _Noise Conditional Score Network_ (NCSN) $s_\theta(x,\sigma)$, which learns the Gaussian-perturbed data distribution $p_{\sigma_i}(x) = \mathbb{E}\_{x' \sim p(x')}\left[p_{\sigma_i}(x\|x')\right] = \int p(x') \mathcal{N}(x; x', \sigma_i^2 I) \mathrm{d} x' $ with various levels of noise $\sigma_i$. $\\{\sigma_i\\}^T_{i=1}$ is a positive geometric sequence that satisfies $\frac{\sigma_1}{\sigma_2}=\frac{\sigma_i}{\sigma_{i+1}}=\frac{\sigma_{T-1}}{\sigma_T}>1$. Since the distributions $\\{p_{\sigma_i}\\}^T_{i=1}$ are all perturbed by Gaussian noise, _their supports span the whole
space and their scores are well-defined, avoiding difficulties from the manifold hypothesis_. To generate samples with annealed Langevin dynamics, we start with the largest noise level $\sigma_0$ and anneal the noise level to $\sigma_T\approx 0$:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    x_{i+1} = x_i + {\alpha_{i+1}} s_\theta(x_i,\sigma_{i+1}) + \sqrt{2\alpha_{i+1}} z_i \label{ALD}\tag{8}
    \end{align*}
$$
</div> 
where $\alpha_{i}=\frac{\gamma}{2} \frac{\sigma_i^2}{\sigma_{i+1}^2}$ is the step-size. The only difference between the above eq(\ref{ALD}) and eq(\ref{Des_LD2}) is that the drift term $s_\theta(x_i,\sigma_{i+1}) \approx -\nabla U_i(x_i)$ is no longer fixed and will change with time.

To learn the score function of the perturbed data distribution, we can use the objective of _denoising score matching_<a href="#sm"><sup>9</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \arg\min_\theta \mathbb{E}_{x_i\sim p_{\sigma_i}(x_i)} \left[||s_\theta (x_i)-\nabla_{x_i} \log p_{\sigma_i}(x_i)||_{2}^{2} \right] 
    = \arg\min_\theta \mathbb{E}_{x,{x}_i \sim p_{\mathrm{data}}(x)p_{\sigma_i}({x}_i|{x})}\left[||s_\theta (x_i)-\nabla_{x_i} \log p_{\sigma_i}(x_i|x)||_{2}^{2}\right], \label{dsm}\tag{9}
    \end{align*}
$$
</div>

From score matching to denoising score matching, the difference is a constant term independent of $\theta$<a href="#smdsm"><sup>11</sup></a>: 
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\mathbb{E}_{x_i\sim p_{\sigma_i}(x_i)}\left[{\mathbb{E}_{x\sim p_{\sigma_i}(x|x_i)}\left[\left||\nabla_{x_i}\log p_{\sigma_i}(x_i|x)\right||^2\right] - \left||\nabla_{x_i}\log p_{\sigma_i}(x_i)\right||^2}\right]
$$
</div>

By matching the score, we are actually minimize the KL divergence between the parameterized and the target (data) probability distribution: $D_\mathrm{KL}(p_{\theta}(x)\|\|p_{data}(x))$.

With the learnt scores $s_\theta(x,\sigma_i) \approx \nabla_x \log p_{\sigma_i}(x)$, we can generate samples according to eq(\ref{ALD}). When $\sigma_i$ is large, modes in $p(x)$ are smoothed out by the Gaussian kernel and $s_\theta(x,\sigma_i)$ points to the _mean_ of the modes; as $\sigma_i$ annealed down, the dynamic will be attracted to the _actual modes_ of the target distribution<a href="#stlmc"><sup>12</sup></a>. 

In the next section, we will see that the sampling algorithm (\ref{ALD}) is indeed a special case of reverse SDEs.
<!-- , which are associated with a deterministic ODE. -->

#### Score-based Stochastic Differential Equations
<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="ULAft">
        <sup>13</sup>Most of the case in ULA, the <em>drift term</em> is time independent and related to the target distribution, unlike the $U_t$ here and the score function in diffusion models.</p>
        <p style='margin-bottom: 10px;' id="PFODE">
        <sup>14</sup><a href="https://openreview.net/forum?id=PxTIG12RRHS">Score-Based Generative Modeling through Stochastic Differential Equations</a>  Y. Song, J. Sohl-Dickstein, D.P. Kingma, A. Kumar, S. Ermon, B. Poole. ICLR 2021.
        </p>
        <p style='margin-bottom: 10px;' id="FM">
        <sup>15</sup>You can find more in <a href="https://openreview.net/forum?id=PqvMRDCJT9t">this paper</a> on flow matching by Yaron Lipman <em>et al</em>.</p>
    </div>
</div>


The continuous SDE form of eq(\ref{ALD}) can be written as:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    d{X_t}=\underbrace{-\nabla U_t(X_t)\mathrm{d}t}_{\text{drift term}} + \underbrace{\sqrt {2}{B_t}\mathrm{d}t}_{\text{diffusion term}},  \quad X_0\sim p_0 \label{ALD2}\tag{10}
    \end{align*}
$$
</div>
where $U_t\propto -\log p_t$ now also depends on time $t$ comparing to $U\propto -\log p_\infty$ in eq(\ref{LD})<a href="#ULAft"><sup>13</sup></a>.

In another wonderful work by Yang Song _et al_<a href="#PFODE"><sup>14</sup></a>, the authors construct forward diffusion process, which maps the target distribution to a (usually) simple distribution, in a more general form of SDEs:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    d{x_t}={f_t(x_t)\mathrm{d}t} + {g_t{B_t}\mathrm{d}t},  \label{SDE}\tag{11}
    \end{align*}
$$
</div>
Each pair of $f_t$ and $g_t$ define the unique forward process and the corresponding $p_t$ given the initial distribution.

<span style="color:blue">From now on, we will swap the notation of time $t$, such that for $t=0$ we have target distribution $p_0 = \pi$ and for $t=T$ we have simple distribution $p_T$.</span>

Furthermore, there exist the corresponding reverse SDE that can be used for sampling/generation:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \mathrm{d}{x_t} &= \left[f_t(x_t) - g_t^2 \nabla_{x_t} \log p_t({x_t})\right]\mathrm{d}t + g_t {\bar{B}_t}\mathrm{d}t, \quad x_0\sim p_0  
    % \\ x_t&=x_{t+1}-f_{t+1}(x_{t+1})+g_{t+1}g^T_{t+1}\nabla_{x_t} \log p_t({x_t})+g_{t+1}z_{t+1} 
    \label{reverseSDE}\tag{12}
    \end{align*}
$$
</div>
This SDE is only meant for time flows backwards from $T$ to 0, and $dt$ is an infinitesimal _negative_ timestep.
Now we can sample from the target distribution use this reverse SDE as long as we have access to $s_\theta \approx \nabla \log p_t$. 
<!-- For $f_t=\nabla_{x} \log p_t({x})$ and $g_t=\sqrt{2}$, we get eq(\ref{ALD2}) as a special case of eq(\ref{reverseSDE}) (note that eq(\ref{reverseSDE}) is reversed in time). -->
<!-- For $f_t=0$ and $g_t=\sqrt{\mathrm{d}[\sigma^2(t)]/\mathrm{d}t}$, we get eq(\ref{ALD}) as a discretization of a special case of eq(\ref{reverseSDE}). -->
<!-- Could we find $f_t$ and $g_t$ such that eq(\ref{ALD2}) as a special case of eq(\ref{reverseSDE}) -->
Note that eq(\ref{ALD2}) and eq(\ref{reverseSDE}) are two different sampling strategies, and we can apply both for sampling<a href="#PFODE"><sup>14</sup></a> (aka. corrector and predictor): we use the corrector to ensure $x_t \sim p_t$ and use the predictor to jump to $p_{t-1}$.

For score-based models (or diffusion models) with forward SDE in the form of eq(\ref{SDE}), we can write the corresponding FP equation in the form of the _continuity equation_<a href="#FM"><sup>15</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \partial_t p_t = -\nabla\cdot(f_t p_t) + \frac{g_t^2}{2}\Delta p_t = -\nabla\cdot((f_t - \frac{g_t^2}{2} \nabla\log p_t) p_t) \label{FP2}\tag{13}
    \end{align*}
$$
</div>
where we define the vector field as $w_t = f_t - \frac{g_t^2}{2} \nabla\log p_t$, $g_t$ is somehow related to the temperature of the stationary distribution, and $p_\infty$ is the initial distribution (not necessarily Gaussian).

The corresponding Ordinary Differential Equations (ODE) of a particle moving along this vector field is the so-called <span style="color:#FFA000"><em>**probability flow ODE**</em></span> (PFODE)<a href="#PFODE"><sup>14</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \mathrm{d} x = \bigg[f_t(x) - \frac{1}{2}g_t^2 \nabla_{x} \log p_t({x})\bigg] \mathrm{d}t \quad \Longleftrightarrow \quad \frac{\mathrm{d}x}{\mathrm{d}t} = v_t \label{prob_ode} \tag{14}
    \end{align*}
$$
</div>
<!-- It not difficult to verify that for the reverse SDE in the form of eq(\ref{reverseSDE}), the corresponding FP equation is still eq(\ref{FP2}). That's to say, we can use this probability flow ODE to travel either forward or backward. -->
And we can use this probability flow ODE to travel either forward or backward in time.

Given $f_t$ and $g_t$ which define the forward diffusion process, we can travel backward in time with learned score function $s_\theta \approx \nabla \log p_t$ or the learned vector field $v_\theta \approx f_t - \frac{g_t^2}{2} \nabla\log p_t$.

<span style="color:gray">
It's worthy noting that, in order to sample from a target distribution according to the reverse SDEs (\ref{reverseSDE}), the score functions with different $t$ are explicitly required (no matter the form of $f_t$ and $g_t$) rather than only the score function of the target distribution.
Moreover, my understanding to the success of diffusion models is that, diffusion models explicitly define paths in both the data space and the probability measure space: given the initial state $x_0$, for each time-step $t$ on the paths, we know where we are ($p_t$) and where we are aiming for ($v_t$).
</span>
<!-- ## Remarks -->
