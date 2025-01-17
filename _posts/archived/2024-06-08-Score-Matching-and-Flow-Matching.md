---
layout: post
title: Score Matching and Flow Matching
categories: Research
description: none
keywords: Score Matching, Flow Matching
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
我感觉是fm从代码实现的角度和diffusion没什么区别，都是先从两个分布分别独立采样x_0 x_1，然后做interpolate，然后预测一个value，这个value也是用x_0和x_1线性组合得到的，如果是x_0就是x_0 prediction，如果是x_1就是\eps prediction，如果是x_0 - x_1 就是velocity prediction

## Langevin Dynamics Sampling 
不过diffusion可以从加噪去噪的角度理解，然后fm可以用cnf的角度去理解。然后cnf那一套东西的优势就是能够算 exact likelihood，就是那个change of variable的公式。实际上对于所有的diffusion 和fm，只要拿到了 probability -flow ODE的表达式（宋飏提出的那个东西）我们就能够算likelihood了。所以我感觉没有必要理解cnf再到fm，如果当下需要先掌握fm的话。

#### Langevin Diffusion
然后我感觉最原始的fm的论文的notation也比较混乱，是lipman和ricky写的[Facepalm]。事实上同时期有rectified flow，stochastic interpolant， action matchin 和 \alpha blending，他们的算法或者代码实现都是一样的，或许你可以都看一眼，理解就深刻一些

<!-- 
**Optimization**: find the minimum $min_{x\in \mathbb{R}^d} U(x)$ | **Sampling**: draw samples from the density $\pi(x)\propto e^{-U(x)}$
 -->

我感觉需要始终牢记一点就是生成模型的范式总是在建立两个分布之间的映射。然后各种各样的方法就是我们如果用一些物理量来描述这个映射。
然后diffusion和fm都可以看作是通过learning的方式去学习 marginal dist p(x_t) 时刻的一个物理量（score, 或者 velocity，或者 eps，或者 x_0）。 因为我们希望能够建立一个连续的过程从分布\pi_0到\pi_1（这也算是diffusion相比gan的一个大的优点，就是多步的性质）。
所以具体做法就是在expectation下面随机独立采样x_0, x_1然后用一个什么schedule来interpolate得到x_t, 然后我们希望我们的nn可以预测正确的t时刻的那个物理量



#### Fokker-Plank Equation
我们前面说的所有的diffusion或者是fm的那个interpolate x_t 预测一个物理量的方式，就是我们可以发现好像diffusion和fm没有区别，至少从算法层面上来看是没有的。
所以我们需要理解一下为什么会有sde和ode两种说法。
首先diffusion因为必须对应一个加噪的过程，也就是要求\pi_1 (或者说\pi_T)必须是standard gaussian，然后diffusion的加噪去噪过程我们可以用一个sde来描述。

这里我们应该区别一下理解diffusion 和 fm的两个出发点，diffusion更多的是存在一个p_data，我们通过一个加噪过程将p_data和 gaussian 联系起来，而fm则是先选择两个distribution， p_data和\pi_1，再用一个没有噪声的schedule (ODE)将他们联系起来（只是作为生成模型的时候大多数 \pi_1就是gaussian)。所以作为生成模型的时候这两个几乎一模一样。

然后fm也可以描述两个数据分布，比如猫和狗的图片，这样的话就和noise无关了，然后那个stochastic interpolant的意思就是可以在fm基础上叠加一个noise的过程，变回一种sde。

总结一下就是，fm和diffusion作为生成模型从结果上没什么区别，因为学的东西是等价的。SDE和ODE的区别感觉主要还是概念上的。毕竟diffusion采样的时候可以用ddpm的sde采样，也可以用ddim或者dpm-solver来求解probabilistic flow ODE （PF-ODE)。
然后fm 采样的时候就直接求解 dx/dt = v_\theta就好了，其实这个dx/dt = v_\theta 就是一个形式极简的 PF-ODE, 所以理论上我们也可以用 sde的方式给fm （当\pi_1是gaussian）采样，就是需要推一下公式

<!-- ## Remarks -->
