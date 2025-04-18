---
layout: post
title: Bayesian Posterior Sampling (1) Introduction
categories: Research
description: none
keywords: Bayesian Posterior, Variational Inference, VAE, MCMC
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

The [last post](2022/06/21/Real-NVP-Intro/) was written around one year ago, when I decided to switch my semester project topic from _style transfer with normalizing flow_ to [_image restoration with diffusion models_](https://yuanzhi-zhu.github.io/DiffPIR/).

In my current engineering-oriented master's thesis, I find myself longing for the elegance of the theory of diffusion models. As a solution, I have made the decision to dedicate my spare time to learning Bayesian sampling (sampling method for bayesian inference).

Hence I will write a series of posts to record this learning process and to improve my understanding of this topic by reorganizing my knowledge. In this post, I will start with some basics and a brief introduction to this topic.

## Introduction

#### Bayesian Inference Problem & Challenging

<p align="center">
  <img src="/images/blog/Bayesian_Posterior_Sampling/Introduction/Bayes.png" alt="Bayes Theorem" width="600">
  <br/>
  <em>Figure 1: Visual Representation of Bayes' Theorem.</em>
</p>

> In the probabilistic approach to machine learning, all unknown quantities — be they predictions about the future, hidden states of a system, or parameters of a model — are treated as random variables, and endowed with probability distributions. The process of inference corresponds to computing the posterior distribution over these quantities, conditioning on whatever data is available.
>
> ------ [_Probabilistic Machine Learning: Advanced Topics_](https://probml.github.io/pml-book/book2.html) by Kevin Patrick Murphy. MIT Press, 2023.

To be specific, we assume the dependence between unknown random latent variable $z$ and the available data $x$ is <span style="color:#FFA000">probabilistic</span> and what we want to do is to estimate $z$ given $x$ with $p(z\| x)$, which we call posterior. 
Most of the time we only have some prior knowledge about $z$ as $p(z)$ and the likelihood model $p(x\|z)$, or equivalently the deep latent variable model $p_\theta(x, z)=p_\theta(z)p_\theta(x\|z)$ through [Bayesian Modeling](https://changliu00.github.io/static/Bayesian%20Learning%20-%20Basics%20and%20Advances.pdf), where $\theta$ can be estimated with maximizing likelihood (**Maximizing the expected log-likelihood is equivalent to minimizing the Kullback-Leibler (KL) divergence between $p_{\mathrm{data}}(\mathbf{x})$ and $p_{\theta}(\mathbf{x})$**):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align}
    -\mathbb{E}_{\mathbf{x}\sim p_{\mathrm{data}}(\mathbf{x})}\left[\log p_\theta(\mathbf{x})\right] &= D_{K L}(p_{\mathrm{data}}(\mathbf{x})\lVert p_{\theta}(\mathbf{x}))\underbrace{-\mathbb{E}_{\mathbf{x}\sim p_{\mathrm{data}}(\mathbf{x})} \left[ \log p_{\mathrm{data}}(\mathbf{x})\right]}_{\text{constant}} \\ &\approx \prod_{n=1}^{N}\log p_\theta(x_{n}) = \prod_{n=1}^{N}\log [\int p_\theta(z_{n})p_\theta(x_{n}|z_{n})dz_{n}] \label{likelihood}\tag{1} \\
    \end{align}
    $$
</div>
where the constant term is the entropy of the data distribution. While the entropy of the empirical data distribution goes to infinity, it is independent of $\theta$ and can be ignored in optimization.

We can compute the posterior $p_{\theta}(z\| x)$ using Bayes's rule (see _Figure 1_ for visual illustration):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align}
    p_{\theta}(z|x)&=\frac{p_{\theta}(x|z)p_{\theta}(z)}{p_{\theta}(x)}\label{bayes}\tag{2} \\
    \end{align}
    $$
</div>
where the evidence term (also called marginal likelihood) served as a normalization constant in the denominator can be formulated as:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align}
    p_{\theta}(x)=\int p_{\theta}(x,z)dz=\int p_{\theta}(x|z)p_{\theta}(z)dz\label{evidence}\tag{3} \\
    \end{align}
    $$
</div>

<!-- Here goes an example of Image Restoration (IR), the task is to estimate the ground truth image $x_0$ give measurement $y$ and we can write the posterior as $p(x_0\|y)=\frac{p(y\|x_0)p(x_0)}{p(y)}$. The prior knowledge $p(x_0)$ may tell us we are observing bird, and the measurement comes from the degradation model $y=\mathcal{H}x_0 +n$ -->

Beyond point estimation (MLE, MAP), we can use the posterior distribution to get posterior expectations of any function $f(z)$, such as mean and marginals. For instance, predicting new output in Bayesian linear regression where $w$ represents the coefficient that we aim to estimate its posterior distribution given the data: $y^\star =\int p(y^\star\|x^\star, w)p(w \| X,y)dw$.

<div class="sidebar" id="intractable">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;'>
        <sup>1</sup>What do we mean by <em>intractable</em> for the evidence term?
        <ul> <li> Curse of dimensionality, complex form</li> <li>Analytical solutions are not available</li> <li>Numerical integration is too expensive</li></ul></p>
    </div>
</div>

However, this integral is usually analytically _intractable_<a href="#intractable"><sup>1</sup></a> to calculate or evaluate, which leads to intractable posterior, and most Bayesian inference requires numerical approximation of such intractable integrals.


#### Two Approaches: Variational Inference & (MCMC) Sampling

In this post, I go through the two primary methodologies utilized to address the problem of Bayesian inference: Variational Inference (VI) and Markov Chain Monte Carlo (MCMC).  I will strive to cover the most significant concepts associated with VI, and also provide a brief introduction to MCMC.

##### Variational Inference

Variational inference (VI) is a method in machine learning that approximates complex probability distributions by finding the most similar, simpler and hence _tractable_ distribution $q(z)$ from a _specified family_ $\mathcal{Q}$, thereby enabling efficient computation and handling of uncertainty.

Most of the time when referring to Variational Inference (VI), we are discussing parametric VI. In parametric VI, we use a parameter $\phi$ to represent the variational distribution $q_\phi(z\|{x})$. 
<span style="color:gray">There is another type of VI called particle-based VI, which utilizes a set of particles $\\{z^{(i)}\\}_{i=1}^{N}$ to represent the variational distribution $q(z\|{x})$.</span>

<p align="center">
  <img src="/images/blog/Bayesian_Posterior_Sampling/Introduction/variational_inference.png" alt="variational_inference" width="600">
  <br/>
  <em>Figure 2: Illustration of Variational Inference.</em>
</p>

The main idea of variational methods is to cast **inference** as an **optimization** problem.
The **goal of VI** is to approximate an intractable probability distribution, so as to find $q_{\phi} \in \mathcal{Q}$ that minimize some discrepancy $D$ (here we use the KL divergence) between $q_{\phi}({z}\|{x})$ and $p_{\theta}({z}\|{x})$:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align}
    q^{\star}_{\phi} = \underset{q_{\phi}\in \mathcal{Q}}{\operatorname{\arg\min }} D_{K L}(q_{\phi}({z}|{x})\lVert p_{\theta}({z}|{x}))\label{KL}\tag{4} \\
    \end{align}
    $$
</div>

The challenge here is that we still don't know the true posterior $p_{\theta}({z}\|{x})$, and the KL divergence is intractable to compute. 
Luckily, we can rewrite the KL divergence in a way that makes it easier to optimize:<a href="#joint_KL"><sup>2</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align*}
    D_{K L}(q_{\phi}({z}|{x})\lVert p_{\theta}({z}|{x}))&=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{q_{\phi}({z}|{x})}{p_{\theta}({z}|{x})}\right]\right] \\ 
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{q_{\phi}({z}|{x})p_{\theta}({x})}{p_{\theta}({x},{z})}\right]\right] \\
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{q_{\phi}({z}|{x})}{p_{\theta}({x},{z})}\right]\right] + \mathbb{E}_{q_{\phi}({z}|{x})}\left[\log p_{\theta}({x})\right]\\
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{q_{\phi}({z}|{x})}{p_{\theta}({x},{z})}\right]\right] + \log p_{\theta}({x})\label{KL_derivation}\tag{5}\\
    \end{align*}
    $$
</div>
where the log evidence $\log p_\theta({x})$ does not change with the choice of $\phi$ during variational inference. 

For convention, we can rewrite this as: 
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align*}
    \log p_\theta({x})=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{p_{\theta}({x},{z})}{q_{\phi}({z}|{x})}\right]\right]+\underbrace{D_{K L}(q_{\phi}({z}|{x})\lVert p_{\theta}({z}|{x}))}_{\geq 0} \label{evidence2}\tag{6}
    \end{align*}
    $$
</div>
Since the KL divergence between $q_{\phi}({z}\|{x})$ and $p_{\theta}({z}\|{x})$ is non-negative, the first term in the RHS of eq(\ref{evidence2}) is a lower bound of the log evidence term $\log p_\theta({x})$, which is named _variational lower bound_, also called the _Evidence Lower BOund_ (ELBO):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$
    \begin{align*}
    \mathcal{L}_{\theta,\phi}({x})=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log p_{\theta}({x},{z})-\log q_{\phi}({z}|{x})\right]\label{ELBO}\tag{7}
    \end{align*}
    $$
</div>

Therefore, the common mission to find the optimal $q_{\phi}({z}\|{x})$ that minimizes the KL divergence (approximates $p_{\theta}({z}\|{x})$) is equivalent to **maximize the ELBO** (without worrying about the evidence term in $p_{\theta}({z}\|{x})$), and we can optimize it w.r.t. both ${\phi}$ and ${\theta}$ (when $\theta$ is unknown) in algorithms such as variational EM.

As $\theta$ is the model parameters, optimizing $\theta$ is essential for learning the underlying model that generates the data. When $\theta$ is tunable, we can jointly optimize $\phi$ and $\theta$ to maximize the ELBO. This joint optimization can help in finding a more accurate posterior approximation ($\phi$) and model parameters that better explain the data ($\theta$ that gives tighter ELBO of likelihood).

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="joint_KL">
            <sup>2</sup>We can also get the same ELMO starting from the KL divergence between joint distributions $D_{KL}(q_{\phi}({x},{z})\lVert p_{\theta}({x},{z}))$, see <a href="https://kexue.fm/archives/5343">this blog</a> by Jianlin Su and <a href="https://blog.alexalemi.com/diffusion.html">this blog</a> by Alex Alemi.</p>
        <p style='margin-bottom: 5px;' id="ELBO">
            <sup>3</sup>It's recommended to read this <a href="https://caseychu.io/posts/perspectives-on-the-variational-autoencoder/">blog</a> by Casey Chu and this famous paper <a href="http://approximateinference.org/accepted/HoffmanJohnson2016.pdf">ELBO surgery</a> by Matthew D. Hoffman and Matthew J. Johnson for more perspectives on the ELBO.
        </p>
        <p style='margin-bottom: 5px;' id="VAE">
            <sup>4</sup>For VAE, there is a great <a href="https://arxiv.org/abs/1906.02691">introduction</a> by D.P. Kingma and Max Welling. 
        </p>
        <p style='margin-bottom: 5px;' id="VAE2">
            <sup>5</sup>For a comprehensive understanding of advanced concepts in VAE such as the $\beta$-VAE, I highly recommend reading Lilian Weng's <a href="https://lilianweng.github.io/posts/2018-08-12-vae/">blog</a>.
        </p>
        <p style='margin-bottom: 5px;' id="DM_loss">
            <sup>6</sup><a href="https://blog.alexalemi.com/diffusion.html">This great blog</a> by Alex Alemi also derives the diffusion loss through variational perspective for those who are interested in diffusion models.
        </p>
    </div>
</div>

We can rewrite the ELBO as follows<a href="#ELBO"><sup>3</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \mathcal{L}_{\theta,\phi}({x}) &= \underbrace{\mathbb{E}_{q_{\phi}({z}|{x})}[\log p_{\theta}({x},{z})]}_{\text{expected log joint}}+\underbrace{\mathbb{H}(q_{\phi}({z}|{x}))}_{\text{entropy}}\label{ELBO2}\tag{8}
    \end{align*}
$$
</div>

Additionally, the ELBO can be reorganized and interpreted as the following in Variational AutoEncoder (VAE)<a href="#VAE"><sup>4</sup></a><sup>,</sup><a href="#VAE2"><sup>5</sup></a>, where $\phi$ and $\theta$ represent the encoder and decoder, respectively:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \mathcal{L}_{\theta,\phi}({x}) &= -[\underbrace{\mathbb{E}_{q_{\phi}({z}|{x})}[-\log p_{\theta}({x}|{z})]}_{\text{expected negative log likelihood}}+\underbrace{D_{K L}(q_{\phi}({z}|{x})\lVert p_{\theta}({z}))}_{\text{KL from posterior to prior}}] \\ &=-\mathbb{E}_{q_{\phi}({z}|{x})}[\underbrace{-\log p_{\theta}({x}|{z})}_{\text{reconstruction error}}+\underbrace{\log q_{\phi}({z}|{x})-\log p_{\theta}({z})}_{\text{regularization (align) terms}}] \label{ELBO3}\tag{9}\\
    \end{align*}
$$
</div>

Suppose $p_{\theta}({x}\|{z})=\mathcal{N}(\mu_\theta(z),\sigma^2)$, and $q_{\phi}({z}\|{x})$ is a deterministic mapping $\psi_{\phi}(x)$. The first term is a _reconstruction error_, proportional to $\|\| x−\mu_\theta(\psi_{\phi}(x))\|\|^2$. And the training objective on given dataset is hence<a href="#DM_loss"><sup>6</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
    $$ 
    \begin{align*}
    \mathbb{E}_{\mathbf{x}\sim p_{\mathrm{data}}(\mathbf{x})}[\mathcal{L}_{\theta,\phi}({x})] = -\mathbb{E}_{\mathbf{x}\sim p_{\mathrm{data}}(\mathbf{x})}\mathbb{E}_{\mathbf{z}\sim q_{\phi}({z}|{x})}[-\log p_{\theta}({x}|{z})+\log q_{\phi}({z}|{x})-\log p_{\theta}({z})] \label{objective}\tag{10}
    \end{align*}
    $$
</div>

The optimization of the above equation (\ref{objective}) usually involve taking gradient w.r.t. $\phi$, which is more difficult as we cannot swap the gradient and the expectation like when taking gradient w.r.t. $\theta$. To resolve this issue, we can use methods like [_score function estimator_](https://mpatacchiola.github.io/blog/2021/02/08/intro-variational-inference-2.html) and the [_reparametrization trick_](https://ermongroup.github.io/cs228-notes/extras/vae/).

##### Markov Chain Monte Carlo
Unlike VI which solves inference with optimization, MCMC tackles it via sampling techniques. More specifically, MCMC applies Monte Carlo methods to generate a sufficient number of samples for an accurate estimation of the posterior distribution. However, it is almost always impossible to directly do so. As a solution, we can use MCMC, which is aimed at simulating a Markov chain whose <span style="color:#FFA000">stationary distribution is $p_{\theta}({z}\|{x})$</span> and hope a <span style="color:#FFA000">fast convergence</span>. 
And guess what, we only need _unnormalized_ probability density (e.g. $p(x,z)$) to simulate the chain! 

**Optimization**: find the minimum $min_{x\in \mathbb{R}^d} U(x)$ | **Sampling**: draw samples from the density $\pi(x)\propto e^{-U(x)}$


<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="gibbs">
            <sup>7</sup>This is called <a href="https://en.wikipedia.org/wiki/Boltzmann_distribution">Boltzmann distribution</a> (him again🤣), which is also called Gibbs distribution.</p>
    </div>
</div>

A more general problem setting is: sampling (=generating new examples) from a target distribution $\pi$ over $\mathbb{R}^d$ whose density is known up to an intractable normalization constant $Z$<a href="#gibbs"><sup>7</sup></a>:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \pi(x) &= \frac{1}{Z}\tilde{\pi}= \frac{\exp(-\beta U(x))}{Z} \label{target_dist}\tag{11}\\
    \end{align*}
$$
</div>
where $\tilde{\pi}$ is the known unnormalized distribution, $\beta$ is an arbitrary positive constant akin to an inverse temperature, and $U(\cdot)$ can be treated as energy function. To make the notation consistent, now we rewrite the problem (\ref{KL}) as:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \pi^\star = \underset{\mu\in \mathcal{P}_2(\mathbb{R}^d)}{\operatorname{\arg\min }} D(\mu \lVert \pi) := \mathcal{F}_\pi(\mu)\label{KL_MCMC}\tag{12} \\
    \end{align*}
$$
</div>
where $D$ is a dissimilarity functional such as KL divergence, and $\mathcal{F}\_{\pi}(\mu)$ is a shorthand of $D(\mu \lVert \pi)$. We can approximate integrals $\int f(\cdot) d\pi$ of any function $f(\cdot)$ with samples from the Markov chain as $\frac{1}{n}\Sigma_{i=b}^{b+n-1}f(x_i)$, where $b,n$ are sufficiently large integers, and $b$ is called the **mixing time** or **burn-in time**. Note that the initial samples from the chain should be discarded because they do not come from the stationary distribution; reducing this is one of the most important factors in securing a fast convergence.

The ultimate goal of this series of posts is exactly to learn various MCMC techniques to sample from $\pi^\star$! \\
It's also highly recommended to try out [this great website](https://chi-feng.github.io/mcmc-demo/) for fantastic MCMC animations first.


The future topics should include:
- MCMC basics
- [Dynamics-based MCMCs](https://changliu00.github.io/static/d_mcmc.pdf)
- Relation to [Wasserstein gradient flows](https://akorba.github.io/resources/Baltimore_July2022_ICMLtutorial.pdf)
  
<!-- [Dynamics-Based MCMCs](https://changliu00.github.io/static/ManifoldSampling-ChangLiu.pdf) 
https://towardsdatascience.com/bayesian-inference-problem-mcmc-and-variational-inference-25a8aa9bce29
file:///C:/Users/admin/Downloads/Zhuo_uchicago_0330D_15243.pdf
https://nlp.jbnu.ac.kr/PGM/slides_other/GibbsSampling.pdf
file:///C:/yuazhu/paper_reading/DeepPM_Draft-2.pdf -->
