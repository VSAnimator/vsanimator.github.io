---
layout: post
title: "Diffusion Models and Policy Gradients Share a Common Structure"
date: 2025-08-06
categories: [diffusion]
tags: [diffusion]
---

*“True enough, this compass does not point north...It points to the thing you want most in this world." -- Jack Sparrow.*

---

When reinforcement learning (RL) researchers compare policy-based and value-based methods, a familiar tradeoff appears: do you learn to evaluate states (or state-action pairs) and derive actions by optimization, or do you learn to act directly?

This distinction—between **learning how to evaluate** and **learning how to act**—shows up again in an unexpected place: **diffusion models**.

In this post, I'll unpack a structural parallel between how **policy gradients outperform Q-learning** in complex environments, and how **diffusion models outperform energy-based models (EBMs)** in high-dimensional generative tasks. The connection isn't just philosophical—it reflects a shared design choice that has practical consequences for stability, scalability, and sample quality.

---

### Learning to Act in Reinforcement Learning

In RL, value-based methods like Q-learning learn a scalar function $Q(s, a)$ that estimates the expected return from taking action $a$ in state $s$. The agent then selects actions by maximizing this function:

$$
\pi(s) = \arg\max_a Q(s, a)
$$

This works well in discrete, low-dimensional settings, but as tasks grow more complex—involving continuous actions or partial observability—the limitations become more pronounced. The inner-loop maximization can be unstable or intractable, and the training signal often lags behind what the agent actually needs to learn.

Policy-based methods instead parameterize a distribution over actions, $\pi_\theta(a \mid s)$, and optimize expected return directly via policy gradients. This bypasses auxiliary value estimates, allows for expressive stochastic policies, and tends to be more robust on complex control problems. It's no accident that algorithms like PPO, TRPO, and SAC have become workhorses for challenging environments where Q-learning struggles.

---

### Diffusion Models and Sampling as Action

A remarkably similar structure emerges when comparing **diffusion models** and **energy-based models** in generative modeling.

EBMs define an unnormalized density:

$$
p(x) \propto \exp(-E(x))
$$

To sample from this distribution, we typically use Langevin dynamics—taking small steps along the gradient of the learned energy:

$$
x_{t+1} = x_t - \eta \nabla_x E(x_t) + \text{noise}
$$

This approach requires training an auxiliary scalar function $E(x)$ whose gradients guide the sampling process. Training is notoriously difficult due to the intractable partition function $Z$, and sampling often suffers from poor mixing and mode collapse in high dimensions.

Diffusion models elegantly sidestep these challenges by learning transition dynamics directly. They define a forward process that progressively adds noise to data, then learn a reverse process that gradually denoises. Each step in the reverse process is parameterized by a neural network—predicting either the original sample, the added noise, or the score $\nabla_x \log p_t(x)$. Sampling proceeds via a learned denoising trajectory:

$$
x_{t-1} = x_t + f_\theta(x_t, t) + \text{noise}
$$

This process is effectively a policy over denoising actions, learned directly rather than derived from an energy landscape.

---

### A Shared Structure

Both policy gradient methods and diffusion models can be viewed as instances of **direct decision modeling** in a sequential process. In RL, the agent learns which action to take given a state. In diffusion, the model learns how to update a noisy sample at each timestep. In both cases, the system learns to produce good transitions without relying on the gradient of an auxiliary function.

| Aspect             | Reinforcement Learning           | Diffusion Models                   |
| ------------------ | -------------------------------- | ---------------------------------- |
| State              | Environment state $s$            | Noisy sample $x_t$                 |
| Action             | Motor command $a$                | Denoising step                     |
| Policy             | $\pi(a \mid s)$                  | Denoising model $f_\theta(x_t, t)$ |
| Auxiliary function | $Q(s, a)$                        | Energy $E(x)$                      |
| Execution          | Follow policy to maximize return | Follow learned denoising path      |

This isn't to say diffusion models are doing RL—there's no reward signal or exploration loop. But structurally, they follow the same logic: define a sequential process and learn the policy that governs it directly.

---

This structural parallel helps explain why diffusion models, like policy-based RL methods, tend to scale better and train more reliably than approaches that rely on learning scalar evaluation functions. When a system must produce a sequence of transitions through a complex space—whether navigating an agent's environment or traversing a model's latent space—it often helps to learn those transitions directly.

The next time you're implementing a diffusion model or training a policy gradient method, consider that you're working with two instances of the same powerful idea: sometimes it's better to learn *what to do* rather than *how good things are*.

Let me know if you'd like to explore how this framing connects to planning, amortized inference, or recent work that *explicitly* treats generative modeling as decision-making.