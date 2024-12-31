---
layout: post
title: New Year Resolution 2025
categories: Research
description: none
keywords: Resolution, Research
mathjax: true
---

<!-- ---
layout: post
title: Pre-PhD Gap 2024
categories: Research
description: none
keywords: Reflection, Research
mathjax: true
--- -->

<style>
    .sidebar {
        float: right; /* Align the sidebar to the right */
        width: 300px; /* Set the width of the sidebar */
        font-family: sans-serif, monospace; /* Example font-family for a light font */
        margin-left: 30px; /* Add margin to the left of the sidebar */
    }
</style>

It's a time to reflect on the past year and set new goals for the next year. 
While I tried to summarize my research journey in 2024, I finally commented out the whole content because it was a bit lengthy and not well-organized, which might not be interesting to read.

But anyway, here comes my new year resolution for 2025.

1. Stay close to my family and friends.
2. Keep healthy and exercise regularly.
3. Do some interesting research (overcome necessary engineering difficulties).
4. Have more discussions with my colleagues and other researchers, inspire more sparks.

<!-- It is at this time, I am about to start my PhD journey at Paris, France. I decided to write a blog post to reflect on my past research experiences of 2024. 
[//]: #  I will also discuss some of the research ideas that I have been thinking about recently.

I really want to emphasis that 2024 is a year I have mixed feelings about.
I graduated from my master's degree officially May this year and finished the master's thesis in last Winter.
What was I doing after the thesis? I was doing a remote internship with Prof. Qiang Liu and his PhD student Xingchao at the University of Texas at Austin.
We started the project in last October at the ICCV 2023 conference as I remember, where I had a remote meeting with Xingchao to show him my preliminary results of the later [SlimFlow](https://github.com/yuanzhi-zhu/SlimFlow).
However, I was too ambitious to do some cool theoretical work and ended up with a lot of dead-end exploration.
The first exploration was about the initialization of the rectified flow, where I simulate backward from data with a trained (on CIFAR10) rectified flow to noise.
By inspecting the noise with PCA and similar tools, I found that the noise data is not Gaussian, but has two modes.
After carefully checking everything, I found that there are certain amount of true data are gray images!!!
We also observed that start from random pure Gaussian noise, the rectified flow can generate a lot of gray images, but with different ratios. This implies that there are potential mode collapse (which might lead to unfair generation) in the rectified flow and diffusion models.
The next step was, now very clear to me, to validate this observation with more datasets and models and to propose a solution to mitigate this issue.

The tough part came when I was trying different strategies to get more balanced data, including use use of all kinds of advanced sampling techniques and training a new small flow model from Gaussian to the multi-modal initial noise.
I tried all kinds of HM algorithms, but none of them worked well in high-dimensional space: the log-likelihood is dominated by some other terms I don't remember (or I forgot on purpose).
The init-flow idea from Qiang was promising in the beginning, but it turned out to be a dead-end as well.
I had an attempt on a human face dataset with faces of different colors, but the observation was not quite consistent with the CIFAR10 dataset.
We were expecting to see multi-modal initial noise data, but the PCA results showed that the minor mode is determined by the background color of the face, which frustrated me a lot. This also made me think the mechanism of diffusion models.
This project was not successful, but I learned a lot from it, especially from Xinachao and Qiang that **I should first validate the hypothesis with toy models as in rectified flow**.
However, now I realized that I should work on ideas that intuitively make sense to me. My motivation should always be doing something that I am interested in and cool. *the target was clear, but the gradient was small*.
I was also not good at pressure control at that time, otherwise I might have made this project into a submission into ICML.
The moment I gave up the init-flow idea I decided to finish the SlimFlow project and do my best to submit it to ECCV. While this is more like a side project, I was luck enough to have it published to ECCV later.
(It is when I started to write this blog post, I found that similar idea can be used to generate structural similar images using similar but different noise, see https://arxiv.org/abs/2412.03895.)
[//]: #  Along this time I was also working on a project aiming to train continuous flow to model the discrete data, we ended up with idea that is close to Hinton's [Analog Bits](https://arxiv.org/abs/2208.04202). But we found that we can train VQVAE with anchor loss.

[//]: #  After this, I spent few time working on continuous modelling of discrete data 
[//]: #  ## Remarks 
After this, I spent more time reading diffusion distillation papers.
Meanwhile, I got reviewed by several professors for my PhD application, and I was lucky enough to have a chance to talk with them.
I got a oral offer from a professor at Polytechnique Paris, and I was planning to move from Zurich to Paris directly in June to start a research engineer position before the official PhD.
However, I was sad to hear that the university or research center did not approve the offer, for some reasons both the professor and I don't know.
This was all of a sudden and I was not prepared for this.

So I was forced to gap for a few months. I had only few weeks to stay in Zurich, and I do not want to apply for a L-type vise to stay in Switzerland for an extra 6 months.
I booked a flight to China and started to look for a internship there. 
I was lucky enough to have a chance to work with Hanshu Yan with the recommendation of Xingchao and Ge Zhang.
This is my first full-time internship and it was quite intense.
The research topic was about efficient visual generation. 
I started with working on PerFlow on PlayGround v2.5.
PerFlow is Hanshu's work on piecewise linear flows to accelerate the inference of diffusion models, and PlayGround is a text-to-image generation model used in production in the company.
This is a bad idea to start with and I should have started with a toy model or baseline model to try out all the improvements then move to the PlayGround model.
Some of the meaningful adjustments I tried are:
1. Instead of simulating the last window close to the clean image, use clean image as the teacher prediction directly.
2. Each time the teacher simulated a batch of windows, train student model several iterations using these window endpoint pairs (similar to defang's neuips work).
3. Add an additional condition w to indicate the current window for the student model, to avoid sharp velocity change at the window terminals.

Unfortunately, none of these tricks help improve the poor results on Playground 2.5...

The first 2 months of my internship teacher me some bitter lessons. On 5.28.2024, I wrote after the group meeting: "作为阶段性总结，实习初期主要的问题是没有围绕核心主线任务扎实的进行实验和思考，导致出现了最后才发现playground模型用默认的sampler 15步就能产生和50步几乎一样的效果，还有直到最后才发现需要2e-4的lr才能让playground v2.5 跳出局部最优。
后续过程中，需要紧紧围绕任务本身，思考以及设计实验。"
I want to really emphasis the importance of baseline and evaluation benchmarks, which tell us the direction of gradient toward our target. (experiment should have clear target, most correct gradient direction)

At the beginning of September, I start to make a table compare the image distillation models and we start to work on Latent Consistency Models and Distribution Matching Distillation.
After getting experience with these methods, we start to, finally, working on video model distillation.

I have to say, many downstream tasks do need a good backbone or base model to work. My initially tried LCM on the Open Sora Plan video diffusion model but it is very difficult to make it work. I believe that distillation require accurate score estimation of the teacher model.
Sometimes you are just jalous of the interns at those big techs.

These practice with distribution matching methods on video diffusion model finally lead to the technique report.

Also in October, I picked an old idea to distill an image restoration model to achieve tunable fiedlity-realism trade-offs. This idea was inspired by Jiatao Gu's BOOT. I still remember I talked to him at ICCV 2023 in Paris about BOOT and asked him when he decide to open source the code because I want to do a follow up work.
The idea itself is very intuitive, I found that the drawbacks of BOOT could be an advantage for image restoration tasks, and I derived a simpler and more effective distillation formula within the rectified flow framework.
Even though the BOOT code is still not released, I managed to finish the distillation code in one night.
What a surprise, I am too lazy I realize. Otherwise, I could have done this much earlier.
The lesson here is to implement code as soon as possible as long as I have an idea, to get the feedback of my intuition as soon as possible.

I want to talk about my understanding of diffusion distillation in the next blog, stay tuned.
(write a review of my intern...
start with all the projects, internal projs
tech reflections one-step model train from scratch (maybe with the help of another model) initialization? 
reflection on video models
reflections on distribution matching loss and regression loss (DMD can be formulated as regression loss?)
DMD is a technique that missed the information of the ode trajectories, which might not be necessary as long as the final distribution is correct and the student can generate in few steps.
many to explore)


I really appreciate the time in Suzhou, Jiangsu with my girl. -->