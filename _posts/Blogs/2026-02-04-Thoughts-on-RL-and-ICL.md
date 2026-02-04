---
layout: post
title: Thoughts on RL and ICL
date: 2026-02-04
categories: Research
description: none
keywords: RL, ICL, Distillation
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

<!-- center the image,  -->

<div style="text-align: center;">
    <img src="/images/blog/RL_ICL/icl_Haruhi.png" alt="RL and ICL" style="max-width: 50%; height: auto;">
</div>
<!-- ![](/images/blog/RL_ICL/icl_Haruhi.png) -->

## A Short Discussion on RL and ICL.

Recently I had a discussion with my friend Jiwen Yu, and I was trying to explain the benefits of RL, which in my opinion, is simply tilting the output distribution of the model towards high-reward regions $p^* = p_{\mathrm{ref}}\exp(\beta r)$.

> "So basically, RL can increase the pass@1, hence the user will get a better answer with higher probability"

> "Yeah, but in practice I found that for my specific question, the LLM will give me much better answers with more details I provided in the prompt."

## Self-Distillation Fine-Tuning (SDFT) and Self-Distillation Policy Optimization (SDPO)

Later I read the paper "Self-Distillation Enables Continual Learning" and I realized that the SDFT is trying to distilling the expert demonstrations in context into the model itself.

Concurrent work, "Reinforcement Learning via Self-Distillation", also proposed a similar idea of using self-distillation to improve the model's performance. To be specific, this time it does not rely on the expert demonstrations but the expert feedback on initial model outputs, then the expert feedback is fed back to the model in the form of context to generate better outputs for self-distillation. This work also demonstrated an elegant approach to utilize text rewards for RL fine-tuning.

## Everything is about Distillation

I was supposed to make this a title for the whole blog, but perhaps it is too clickbaity. 
Nevertheless, here I would like to emphasize that the core idea is, for every method that is scalable at test time (with the help of external reward models or expert signals $c_\mathrm{ext}$), we can always find a way to distill the knowledge back into the model itself, so that at test time, the model can perform better without relying on external signals. 

From another perspective from the first principal by Rishabh Agarwal on twitter, "Teacher doesn't have to be a bigger neural net, just something better than the student".

If we formulate the generation process as $p_\theta(x\mid c)$, where $c$ is the context, then we can always find a better teacher distribution of the same model $p(x\mid c, c_\mathrm{ext})$ that leverages the external signals $c_\mathrm{ext}$ to provide better outputs. Then we can distill the knowledge from the teacher back to the student model by minimizing some divergence between the two distributions, e.g. KL divergence.

Some examples include:
- RL: distill the reward model and the policy into the model itself.
- SDFT: distill the expert demonstrations in context into the model itself.
- SDPO: distill the expert feedback as text reward in context into the model itself.

To end, everything is about distillation!

## Acknowledgements
The author thanks Jiwen Yu and Yao Teng for their valuable feedbacks (for me to distill my thoughts).