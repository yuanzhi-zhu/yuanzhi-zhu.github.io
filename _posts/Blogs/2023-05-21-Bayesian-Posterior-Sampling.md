---
layout: post
title: Basysian Posterior Sampling (1) Introduction
categories: Research
description: none
keywords: Computer Vision, Basysian, Posterior Sampling
mathjax: true
---


The [last post](2022/06/21/Real-NVP-Intro/) was written around one year ago, when I decided to switch my semester project topic from style transfor with normalizing flow to applications (image restoration) of diffusion models.

In my current engineering-oriented master thesis, I find myself longing for the elegance of theory of diffusion models. As a solution, I have made the decision to dedicate my spare time to learning Bayesian sampling.

Hence I will write a series of posts to record this learning process and to improve my understanding of this topic by reorganizing the knowledge. In this post, I will start with some basics and a brief introduction of this topic.

## Introduction

#### Bayesian Inference Problem & Challenging

<p align="center">
  <img src="/images/blog/Bayesian_Posterior_Sampling/Introduction/Bayes.png" alt="Bayes Theorem" width="700">
  <br/>
  <em>Figure 1: Visual representation of Bayes' Theorem.</em>
</p>

> In the probabilistic approach to machine learning, all unknown quantities — be they predictions about the future, hidden states of a system, or parameters of a model — are treated as random variables, and endowed with probability distributions. The process of inference corresponds to computing the posterior distribution over these quantities, conditioning on whatever data is available.
>
> ------ [_Probabilistic Machine Learning: Advanced Topics_](https://probml.github.io/pml-book/book2.html) by Kevin Patrick Murphy. MIT Press, 2023.

To be specific, we assume the dependence between unknown random latent variable $z$ and the available data $x$ is <span style="color:#FF8C00">probabilistic</span> and what we want to do is to estimate $z$ given $x$ with $p(z\| x)$, which we call posterior. 
Most of the time we only have some prior knowledge about $z$ and the likelihood model $p(x\|z)$ (or $p_\theta(x, z)$ through [Bayesian Modeling](https://changliu00.github.io/static/Bayesian%20Learning%20-%20Basics%20and%20Advances.pdf)), we can compute the posterior $p(z\| x)$ using Bayes's rule (see _Figure 1_ for visual illustration):
<div style="overflow-x: auto; white-space: nowrap;">
  $$p(z|x)=\frac{p(x|z)p(z)}{p(x)}$$
</div>
where the evidence term (also called marginal likelihood) served as a normalization constant in the denominator can be formulated as:

$$ p(x)=\int p(x,z)dz=\int p(x|z)p(z)dz $$

<!-- Here goes an example of Image Restoration (IR), the task is to estimate the ground truth image $x_0$ give measurement $y$ and we can write the posterior as $p(x_0\|y)=\frac{p(y\|x_0)p(x_0)}{p(y)}$. The prior knowledge $p(x_0)$ may tell us we are observing bird, and the measurement comes from the degradation model $y=\mathcal{H}x_0 +n$ -->

Beyond point estimation (MLE, MAP), we can use the posterior distribution to get posterior expectations of function of $z$, such as mean and marginals.

However, this integral is usually analytically _intractable_ to calculate or evaluate, which lead to intractable posterior, and most Bayesian inference requires numerical approximation of intractable integrals.


<div style="font-size: 12px;">
What do we mean by <em>intractable</em> for the evidence term?
<ul> <li> Curse of dimensionality, complex form</li> <li>Analytical solutions are not available</li> <li>Numerical integration is too expensive</li>
</ul>
</div>

#### Two Approaches: Variational Inference & (MCMC) Sampling

In this post we will briefly go through the two main methods that can be used to tackle the Bayesian inference problem: Variational Inference and MCMC.

##### Variational Inference

Variational inference (VI) is a method in machine learning that approximates complex probability distributions by finding the most similar, simpler and hence _tractable_ distribution $q(z)$ from a _specified family_ $\mathcal{Q}$, thereby enabling efficient computation and handling of uncertainty.

Most of the time when referring to Variational Inference (VI), we are discussing parametric VI. In parametric VI, we use a parameter $\phi$ to represent the variational distribution $q_\phi(z)$. 
<span style="color:gray">There is another type of VI called particle-based VI, which utilizes a set of particles ${z^{(i)}}_{i=1}^{N}$ to represent the variational distribution $q(z)$.</span>

For any choice of inference model $q_{\phi}({z}\|{x})$, including the choice of variational parameters $\phi$, we have:

<div style="overflow-x: auto; white-space: nowrap;">
    $$
    \begin{align*}
    \log p_\theta({ x})&=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log p_\theta({x})\right]\\ 
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{p_\theta({x},{z})}{p_\theta({z}|{x})}\right]\right] \\
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[{\frac{p_{\theta}({x},{z})}{q_{\phi}({z}|{x})}}{\frac{q_{\phi}({z}|{x})}{p_{\theta}({z}|{x})}}\right]\right] \\
    &=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{p_{\theta}({x},{z})}{q_{\phi}({z}|{x})}\right]\right]+\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log\left[\frac{q_{\phi}({z}|{x})}{p_{\theta}({z}|{x})}\right]\right]
    \end{align*}
    $$
</div>
where the first term is _variational lower bound_, also called the _evidence lower bound_ (ELBO):

$$
\mathcal{L}_{\theta,\phi}({x})=\mathbb{E}_{q_{\phi}({z}|{x})}\left[\log p_{\theta}({x},{z})-\log q_{\phi}({z}|{x})\right]
$$

and the second term is the Kullback-Leibler (KL) divergence between $q_{\phi}({z}\|{x})$ and $p_{\theta}({z}\|{x})$, which is non-negative: 

$$D_{K L}(q_{\phi}({z}|{x})\lVert p_{\theta}({z}|{x})) \geq 0$$

The common goal is to maxmize the ELBO, that is, to find the optimal $q_{\phi}({z}\|{x})$ that minimizes the KL divergence (approximates $p_{\theta}({z}\|{x})$). Additionally, the ELBO can be reorganized as the following in Variational Autoencoder (VAE):

$$
\mathcal{L}_{\theta,\phi}({x};z)=\mathbb{E}_{q_{\phi}({z}|{x})}[\underbrace{\log p_{\theta}({x}|{z})}_{\text{Negative reconstruction error}}+\underbrace{\log p_{\theta}({z})-\log q_{\phi}({z}|{x})}_{\text{Regularization (align) terms}}]
$$

<div style="font-size: 12px;">
<p style='margin-bottom: 10px;'>
For VAE, there is a great <a href="https://arxiv.org/abs/1906.02691">introduction</a> by D.P. Kingma and Max Welling.</p>
</div>

##### Markov Chain Monte Carlo
We can also use Monte Carlo method to draw enough samples to estimate the posterior. However, it is almost always impossible to directly do so. As a solution, we can use Markov Chain Monte Carlo (MCMC), which aimed at simulating a Markov chain whose stationary distribution is the target posterior densities. And guess what, we only need _unnormalized_ probability density (e.g. $p(x,z)$) to simulate the chain!

The ultimate goal of this series of posts is to learn various MCMC techniques exactly!

The future topics should include:
- MCMC basics
- [Dynamics-Based MCMCs](https://changliu00.github.io/static/d_mcmc.pdf)
- relation to diffusion models
  
<!-- [Dynamics-Based MCMCs](https://changliu00.github.io/static/ManifoldSampling-ChangLiu.pdf) 
https://towardsdatascience.com/bayesian-inference-problem-mcmc-and-variational-inference-25a8aa9bce29
file:///C:/Users/admin/Downloads/Zhuo_uchicago_0330D_15243.pdf
https://nlp.jbnu.ac.kr/PGM/slides_other/GibbsSampling.pdf
file:///C:/yuazhu/paper_reading/DeepPM_Draft-2.pdf -->
