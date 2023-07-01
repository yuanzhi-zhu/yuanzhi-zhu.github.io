---
layout: post
title: Bayesian Posterior Sampling (2) MCMC Basics
categories: Research
description: none
keywords: Bayesian, Acceptance-Rejection, MCMC, Metropolis-Hastings
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

In the previous post, I showed that we can use MCMC methods to draw samples from a target distribution $\pi$. 
In this post, I will present the most classical MCMC method, _Metropolis-Hastings_ (MH) and its variants.
Prior to MCMC methods, I want to first get the readers familiar with another Monte Carlo approach named [_acceptance-rejection method_](https://en.wikipedia.org/wiki/Rejection_sampling), which is also able to generate samples from a (unnormalized) distribution $\tilde\pi$ and whose shortcomings motivate us to use Markov chain.


## Acceptance-Rejection Method

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 10px;' id="easy_sampling">
        <sup>1</sup>Anyway, in order to get samples from $\pi(x)$ we must resort to sampling, right?</p>
        <p style='margin-bottom: 10px;' id="analog">
        <sup>2</sup>Recall how we use the Monte Carlo method to estimate the value of $\pi$? This process involves the decision of disregarding all darts that fall outside of a circle region!</p>
    </div>
</div>

To sample directly from the target distribution $\pi(x)$ is unfeasible.
The technique named acceptance-rejection sampling, also known as rejection sampling, provides a solution to this issue. 
The fundamental concept here is to propose a different probability distribution, $G$, with a density function $g(x)$, which not only have an **efficient sampling algorithm**<a href="#easy_sampling"><sup>1</sup></a> readily available, but should also span (at least) the same domain as $\tilde\pi(x)$ (Otherwise, there would be parts of the curved area we want to sample from $\pi(x)$ that could never be reached.).
Consequently, it's seem natural now to construct a **decision rule**<a href="#analog"><sup>2</sup></a> to determine whether to accept samples from the proposed distribution $g$, of course based on our knowledge of $\tilde\pi(x)$. This decision rule should ensure that the long-term fraction of time spent in each state is proportional to $\tilde\pi(x)$.


Let first have a look at the algorithm, which is quite simple by repeating the following two steps until enough samples are accepted:

> 1. sample $x$ from proposal $g$.
> 2. accept with probability $p_{\mathrm{accept}}(x) = \frac{\tilde\pi(x)}{Mg(x)}$.

where $M$ is a sufficient large constant to ensure $\left[\frac{\tilde\pi(x)}{Mg(x)}\right]$ can be interpreted as probability, that is $0 < p_{\mathrm{accept}} < 1$ for all values of $x$. This also require $g(x)>0$ whenever $\tilde\pi(x)>0$ as mentioned above.
<!-- 
Saying now we have a sample from $g$, the intuition is that we would like to take this sample if it is highly likely under the distribution $\tilde\pi(x)$, and reject it if otherwise. On the other hand, we also want to consider whether this sample is a likely to occur again under the proposal distribution $g$. -->

Let's now establish that the accepted samples indeed come from the distribution $\pi$. Suppose that the act of accepting a sample is denoted as $A$. Therefore, for any given sample $s$, we can state that:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    p(s|A) &= \frac{p(A|s)p(s)}{p(A)} = \frac{p_{\mathrm{accept}}(s)g(s)}{\int p_{\mathrm{accept}}(x)g(x) dx} = \frac{\tilde\pi(s)/M}{\int \tilde\pi(x)/M dx} = \frac{\tilde\pi(s)}{\int \tilde\pi(x) dx} = \pi(s) \label{AR}\tag{1}
    \end{align*}
$$
</div>

The denominator $p(A)$ is the unconditional acceptance probability and is equal $\frac{Z}{M}$ (For a more thorough explanation, please refer to the detailed derivation available at this [wiki](https://en.wikipedia.org/wiki/Rejection_sampling#Theory)):
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    p(A) = \int p(A|x)p(x)dx = \int \frac{\tilde\pi(x)}{M} dx = \frac{Z}{M} \label{denominator}\tag{2}
    \end{align*}
$$
</div>

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="curse">
            <sup>3</sup>The <em>curse of dimensionality</em> indeed refers to <strong>various</strong> counterintuitive phenomena in high dimension. Have a look at the first lecture of <a href="https://www.imo.universite-paris-saclay.fr/~christophe.giraud/Orsay/HDPS.html">High-dimensional Statistics & Probability</a> given by Christophe Giraud.</p>
    </div>
</div>

The unconditional acceptance probability $p(A)$ tell us, we need on average $\frac{M}{Z}$ draws to get an accepted sample, which can be very inefficient when $M$ is very large. And especially when we have random variable $x$ in high dimension, rejection sampling suffers from the _"curse of dimensionality"_<a href="#curse"><sup>3</sup></a>.
<!-- And there are several methods with better proposal $g$ to alleviate this issue. -->

Moreover, in acceptance-rejection method, each draw is independent and information from the last draw is discarded completely (i.e, we may want to explore the its neighbor if the sample from last draw get accepted). It's natural to consider adding some correlation between the closest samples, and this is when Markov chain (or the Markov transition kernel $p(x'\|x)$) is introduced.

## Recap of Markov Chain
A (discrete-time) Markov chain is a sequence of random variables $X_1, X_2, X_3, ...$ with the Markov property:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    p(X_{t}=x_{t}| X_{t-1}=x_{t-1},\dots ,X_{0}=x_{0})=p(X_{t}=x_{t}| X_{t-1}=x_{t-1}) \label{Markov_property}\tag{3}
    \end{align*}
$$
</div>
and a Markov process is _uniquely_ defined by its transition probabilities $p(x'\| x)$.

In our case, we always want the Markov chain to have a _unique_ _stationary_ distribution that is indeed $\pi$.

**Markov Chain Properties**
- Prior $p(X_1)$ and transition probabilities $p(X_{t+1} \| t_n )$ independent of $t$.
- _Ergodic_ Markov Chains: there exists a finite $t$ such that every state can be reached from every state in exactly $t$ steps.
- Stationary Distributions: $\lim\limits_{T \rightarrow \infty}  p\left(X_{T}=x\right)=\pi(x)$, which is independent of $p(X_1)$.


<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="detailed_balance">
            <sup>4</sup>The principle of detailed balance was explicitly introduced for collisions by our favorite <a href="https://en.wikipedia.org/wiki/Ludwig_Boltzmann">Ludwig Boltzmann</a>.</p>
    </div>
</div>


**Detailed Balance Equation**

A chain satisfies detailed balance<a href="#detailed_balance"><sup>4</sup></a> if we have:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    \pi(\mathbf{x}) p\left(\mathbf{x}^{\prime} | \mathbf{x}\right)=\pi\left(\mathbf{x}^{\prime}\right) p\left(\mathbf{x} | \mathbf{x}^{\prime}\right) \label{Detailed_Balance}\tag{4}
    \end{align*}
$$
</div>
This means in the in-flow to state $x'$ from $x$ (the probability of being in state $x$ times the transitional probability from $x$ to $x'$) is equal to the out-flow from state $x'$ back to $x$, and vice versa (Or each elementary process is in equilibrium with its reverse process at equilibrium / stationary). If a chain with transitional probability $p\left(\mathbf{x}^{\prime} \| \mathbf{x}\right)$ satisfies the above detailed balance, then $\pi(\mathbf{x})$ is its stationary distribution. And it worth to mention that detailed balance is in general sufficient but not necessary condition for stationarity.

## Metropolis-Hastings Methods
In the previous section, we learnt that the key of rejection sampling is <span style="color:#FFA000">**proposal**</span> and <span style="color:#FFA000">**decision rule**</span>. These principles remain applicable in MH techniques.
In MH methods, instead generate samples independently from proposal $g$, the proposal (with Markov property) $g(x_{t+1}\|x_t)$ generate new candidate based on the current sample value (The samples are correlated.).

Again, let have a look at the algorithm first, which is similar to rejection sampling. For each current state $x_t$:

> 1. sample $x'$ from proposal $g(x'\|x=x_t)$.
> 2. accept $x_{t+1}=x'$ with probability $p_{\mathrm{accept}}(x') := p_{\mathrm{accept}}(x'\|x)$, set $x_{t+1}=x_{t}$ with probability $1-p_{\mathrm{accept}}(x')$.

where the probability $p_{\mathrm{accept}}(x')$ is defined as:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
    p_{\mathrm{accept}}(x') = \min (1,\alpha) =\min \left(1, \frac{\tilde\pi\left(x^{\prime}\right) g\left(x | x^{\prime}\right)}{\tilde\pi(x) g\left(x^{\prime} | x\right)}\right) \label{accept_probability}\tag{5}
    \end{align*}
$$
</div>

The transition probability of the Markov chain defined in the MH algorithm can be written as: 
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        p(x'|x)=\begin{cases}
        g(x'|x)p_{\mathrm{accept}}(x'|x), & \text{if } x'\neq x \\
        g(x|x)+\sum_{x'\neq x}g(x'|x)(1-p_{\mathrm{accept}}(x'|x)), & \text{otherwise}
        \end{cases} \label{transition_kernel}\tag{6}
    \end{align*}
$$
</div>

We need to ascertain that our proposal satisfies the detailed balance equation (\ref{Detailed_Balance}). To accomplish this, we can verify (left as exercise) that the left and right sides of the equation are equal using the target distribution $\pi$ and the transition probability $p(x'\|x)$, which is defined in (\ref{transition_kernel}).


#### Rejection Sampling as Special Case
When the proposal $g(x'\|x)$ is independent of previous state $x_t$ — in other words, $g(x'\|x) = g(x')$ is an independence sampler — we end up with the method rejection sampling introduced in the beginning.

<div class="sidebar">
    <div style="font-size: 12px;">
        <p style='margin-bottom: 5px;' id="RWMH">
            <sup>5</sup>This should be indeed called Random Walk Metropolis, as the <i>Hastings correction</i> will be introduced when the proposal $g(x'|x)$ is asymmetric.</p>
        <p style='margin-bottom: 5px;' id="HD">
            <sup>6</sup>You can test your intuition of high-dimension in lecture note 1, and check the argument why HM is inefficient in high dimension in note 4 from <a href="https://canvas.stanford.edu/courses/66218">this course</a>.</p>
        <p style='margin-bottom: 5px;' id="HD2">
            <sup>7</sup><a href="https://elevanth.org/blog/2017/11/28/build-a-better-markov-chain/">This blog</a> by Richard McElreath present a demo on why MH fails in high dimension and motivate the usage of Hamiltonian Monte Carlo.</p>
        <p style='margin-bottom: 5px;' id="HD3">
            <sup>8</sup><a href="https://betanalpha.github.io/assets/case_studies/probabilistic_computation.html">This blog</a> and <a href="https://arxiv.org/abs/1701.02434">this paper</a> by Michael Betancourt studies counterintuitive behaviors of high-dimensional spaces which can explain why Metropolis does not scale well enough to high dimensions.</p>
    </div>
</div>

#### Random Walk Metropolis-Hastings<a href="#RWMH"><sup>5</sup></a>
When the proposal $g(x'\|x)$ is Gaussian, we get random walk Metropolis-Hastings, where the new state is a random walk based on the previous state. This proposal can be written as following:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        g(x'|x) = \mathcal{N}(x';x,\tau^2\mathbf{I}) \label{Random_Walk}\tag{7}
    \end{align*}
$$
</div>
where $\tau$ is a scale factor chosen to facilitate rapid mixing, often referred to as the random walk step size. This step size plays a crucial role in balancing the trade-off between exploration, which is essential for covering the distribution, and maintaining an acceptable acceptance rate. 

> [RR01b](https://projecteuclid.org/journals/statistical-science/volume-16/issue-4/Optimal-scaling-for-various-Metropolis-Hastings-algorithms/10.1214/ss/1015346320.full) prove that, if the (target) posterior is Gaussian, the asymptotically optimal value is to use $\tau^2 = 2.382/D$, where $D$ is the dimensionality of $x$; this results in an acceptance rate of 0.234, which (in this case) is the optimal tradeoff ...
>
> ------ [_Probabilistic Machine Learning: Advanced Topics_](https://probml.github.io/pml-book/book2.html) by Kevin Patrick Murphy. MIT Press, 2023.

Since the proposal $g(x'\|x)$ is now symmetric with $g(x'\|x)=g(x\|x')$, 
we have simplified $p_{\mathrm{accept}}(x') = \min \left(1, \frac{\tilde\pi\left(x^{\prime}\right)}{\tilde\pi(x)}\right)$. This simplified acceptance probability is more easily understood: if $x'$ is more probable than $x$, we unquestionably transition there, otherwise, we may still move there anyway by change, depending on the relative probabilities.

However, in high-dimensional space<a href="#HD"><sup>6</sup></a>, this classical MH algorithm is not applicable as it will end up without moving at all for a long time<a href="#HD2"><sup>7</sup></a><sup>,</sup><a href="#HD3"><sup>8</sup></a>.

<!-- This area of probability mass is a narrow surface that lies far from the mode and is called the typical set.  -->

#### Metropolis-Adjusted Langevin Algorithm (MALA)
The well-known Metropolis-Adjusted Langevin Algorithm, which we will revisit in the next post, can be considered as MH with improved proposals:
<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$ 
    \begin{align*}
        g(x'|x) = \mathcal{N}(x';x-\tau \nabla \tilde{\pi}(x),2\tau I) \label{MALA}\tag{8}
    \end{align*}
$$
</div>
where the additional term $\tau \nabla \tilde{\pi}(x)$ drives the samples toward areas with high probability density, which makes the proposed samples more likely to be accepted.

## Remarks

In this post we mainly discussed the basic idea of MCMC method, especially the _**propose-accept scheme**_ in Metropolis-Hastings algorithm. 
In the next posts, we will explore a more intuitive family of methods, the dynamic-based MCMC.