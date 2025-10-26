---
layout: post
title: A Second-order MeanFlow
categories: Research
description: none
keywords: Resolution, Research
mathjax: true
authors:
  - "Yuanzhi Zhu"
  - "Runlong Liao"
---

<style>
    .sidebar {
        float: right; /* Align the sidebar to the right */
        width: 300px; /* Set the width of the sidebar */
        font-family: sans-serif, monospace; /* Example font-family for a light font */
        margin-left: 30px; /* Add margin to the left of the sidebar */
    }
</style>


# Second-order MeanFlow: Combining Forward and Backward Distillation
 
In this blog, We show how the MeanFlow (backward distillation) identity and the tri-consistency constraint can be combined to produce a *second-order* identity for the student velocity in diffusion/flow distillation. 
Similar to MeanFlow, this 2-order MeanFlow contains remarkably simple and elegant target: the average of two end point velocities and a second-order correction term. 
<!-- Below we present the derivation, explain the approximations, show how to compute the required time derivatives in practice (JVP / autograd), and give practical loss functions and implementation suggestions. -->

---

## 1. Setup: Notation and Goals

- $x_t$ denotes the state at time $t$ on the diffusion/flow trajectory.  
- $v_\phi(x_t,t)$ is the *pre-trained teacher* velocity field (teacher's instantaneous velocity at $(x_t,t)$).  
- $v_\theta(x_t,t,s)$ is the *student* average velocity intended to take a sample at time $t$ to the time $s$ in a *single step* (a short-cut/one-step generator). We treat $s > t$.  
- We will often use small increments $ds$ with $s_2 = s_1 + ds$.

Two consistency principles underly the derivation:

1. **MeanFlow (backward) identity** (backward distillation): for $s > t$,

   <div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
   $$
   v(x_t,t,s) = v_\phi(x_t,t) + (s-t)\frac{d}{dt}\bigl[v(x_t,t,s)\bigr],
   $$
   </div>
   
   which expresses the student average velocity from timestep $t$ to $s$ in terms of the (teacher's) instantaneous velocity at $t$ and the time derivative of the student velocity with respect to the local timestep $t$.

2. **Tri-consistency** (additivity of short segments): for $s_2 > s_1 > t$,
   
   <div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
   $$
   (s_1-t)\,v(x_t,t,s_1) + (s_2-s_1)\,v(x_{s_1},s_1,s_2) \;=\; (s_2-t)\,v(x_t,t,s_2).
   $$
   </div>
   
   This says that two short forwards ($t$ to $s_1$ to $s_2$) should equal the single forward (from $t$ to $s_2$).

**Goal:** derive a practical target for $v_\theta(x_t,t,s)$ that consider both the forward and backward constraints.

---

## 2. A Intuitive Derivation

Start from tri-consistency and replace the second segment ($s_1$ to $s_2$) with teacher PF-ODE simulation:

<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
(s_1-t)\,v(x_t,t,s_1) + \mathrm{ODE}\bigl[v_\phi,x_{s_1},s_1,s_2\bigr] = (s_2-t)\,v(x_t,t,s_2).
$$
</div>

For small step $ds = s_2 - s_1$ the one-step Euler approximation to the teacher ODE gives

<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\mathrm{ODE}[v_\phi,x_{s_1},s_1,s_2] \approx \,v_\phi(x_{s_1},s_1) ds.
$$
</div>

Now set $s_1 = s$ and $s_2 = s + ds$. Substitute the MeanFlow identity (backward) for the two student velocities $v(x_t,t,s)$ and $v(x_t,t,s_2)$. After algebra and cancellation of terms proportional to $ds$, we arrive at the following mixed relation:

<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
v_\phi(x_s,s) - v_\phi(x_t,t)
\;=\;
2(s-t)\,\frac{d}{dt}v(x_t,t,s) + (s-t)^2\;\frac{d^2}{dt\,ds}v(x_t,t,s).
$$
</div>

We can substitute the MeanFlow identity to eliminate the first time derivative and rearrange to isolate $v(x_t,t,s)$, yields the **second-order MeanFlow identity**:

<div style="overflow-x: auto; white-space: nowrap; margin-top: -20px;">
$$
\boxed{\;v(x_t,t,s) = \frac{1}{2}\bigl(v_\phi(x_t,t) + v_\phi(x_s,s)\bigr)
- \frac{1}{2}(s-t)^2\,\frac{d^2}{dt\,ds}v(x_t,t,s)\;}
\tag{★}
$$
</div>

**Remarks**
- Equation (★) is the second-order MeanFlow expression: the student velocity equals the average of the two teacher velocities plus a second-order correction involving the mixed partial $\dfrac{d^2}{dt\,ds}v$.
- (Quick validation) Similar to MeanFlow, when $t=s$, Equation (★) degrades to flow matching loss.
- Unlike forward and backward distillation loss, Equation (★) can not be simply interpreted as a special case of tri-consistency.
- In practive, we expect to use this loss along with the first-order MeanFlow loss to improve the few-step performance (requires more computation).
