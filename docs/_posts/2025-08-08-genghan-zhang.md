---
layout: post
title: "Building Self-Improving LLM Agents: Paths Toward Lifelong Learning"
date: 2025-08-09
categories: [llm_agents]
tags: [llm_agents]
---

*Blog post authored jointly by Vishnu Sarukkai and Genghan Zhang*

Large Language Models (LLMs) are remarkable at recalling what they know. With a few carefully chosen examples, they can adapt in context to new tasks. But when deployed as autonomous agents, they rarely get better with experience. Each task is solved as if the last one never happened.

Our two recent research papers, developed independently but exploring a common question, challenge this status quo:

> **Can LLM agents improve themselves during deployment, simply by remembering and reusing their own successful solutions?**

One paper, [Self-Generated In-Context Examples Improve LLM Agents for Sequential Decision-Making Tasks](https://arxiv.org/abs/2505.00234), shows that LLM agents can boost their performance on sequential decision-making tasks by learning in-context from their attempts at previous tasks. The other, [Adaptive Self-Improvement LLM Agentic System for ML Library Development](https://arxiv.org/abs/2502.02534), applies similar principles to a very different domain: building machine learning operator libraries in the architecture-specific programming language [STeP](https://ppl.stanford.edu/papers/YARCH24_STEP.pdf). 

In this post, we’re collaborating to draw out the common principles, the key differences, and what they tell us about how AI agents might one day learn continuously—not just in pretraining, but throughout their lifetimes.

## The Shared Idea: Experience as Data

Both works start from the same observation: LLMs are strong in-context learners, but they rely on examples provided upfront. What if those examples came from the agent itself?

The self-improvement loop looks like this:

1. The agent attempts a task, reasoning step by step or generating a candidate solution.
2. A verifier checks whether the attempt was successful.
3. Successful solutions are stored in a structured database.
4. When a new task arrives, relevant past successes are retrieved and added to the context, improving the next attempt.

This simple feedback loop doesn’t change the model’s weights. It doesn’t need a curated dataset. It relies on **autonomously generated, verified experience**. Over time, each agent builds its own **task-specific knowledge base**, improving its behavior without external supervision.

Both papers explore how far this principle can take an LLM agent. The settings—and therefore the challenges—are very different, but the recipe is shared.

## Two Domains, Two Perspectives

### Sequential Decision-Making

The first paper tests self-improvement in multi-step reasoning environments like ALFWorld, InterCode, and Wordcraft. Tasks involve planning sequences of actions under partial observability and sparse rewards.

Here, trajectories are **sequences of reasoning and action steps from a ReAct agent**. Success is determined by environment feedback: did the agent reach the goal state? The challenge is to curate demonstrations that are not only successful but also **diverse and complementary**, covering different strategies for related problems. Over time, the agent’s growing library of successes makes it more reliable at solving unseen tasks of the same type.

### ML Operator Library Development

The second paper offers an adaptive self-improvement algorithm for writing ML operator code in low-level architecture-specific programming languages. The agent generates candidate implementations, compiles and tests them, and keeps only correct and high-quality solutions.

The challenges here are different. The search space is effectively infinite, with only sparse islands of correctness. Verification is strict and deterministic: a program either compiles and passes tests or it doesn’t. Retrieval requires deduplication (to avoid storing near-identical solutions) and stratification (prioritizing hard-earned experiences). Crucially, the results are not just better agent behavior—they form a **persistent, reusable ML library** that can benefit future systems.

## Shared Principles, Different Pressures

The two papers, developed independently, converged on the same core principles:

1. **Experience compounds:** Each success is not just an endpoint but a resource for future problem solving.
2. **Autonomy matters:** Agents can improve without curated examples or retraining, using only verifier feedback.
3. **Context is dynamic:** In-context learning doesn’t have to be static; the context itself can evolve with experience.
4. **Domain adaptation is possible:** Agents can specialize to the task distribution they actually face during deployment.

But the domains place different pressures on these principles. In sequential decision-making, the difficulty is reasoning over long horizons and selecting complementary examples. In ML library building, the bottleneck is discovering rare correct solutions in a vast code space and assembling them into a durable artifact. The same idea—self-generated, self-curated context—adapts to both worlds, but the algorithms around it look different.

## A Glimpse of Lifelong Learning

These two papers suggest that we don’t always need larger models or more offline data to make agents smarter. Sometimes, it’s enough to give them the ability to **remember their own successes.**

Instead of treating deployed LLMs as static artifacts of pretraining, we can imagine agents that:
1. **Invent and verify solutions on the fly.**
2. **Curate their own knowledge bases,** specialized to the environments they inhabit.
3. **Get better with every solved task,** building lasting expertise over time.

In sequential decision-making, that expertise is a growing set of reasoning traces. In ML library development, it’s an expanding repository of verified operator code. In both cases, competence compounds—autonomously, continually, and without human supervision.

## Looking Forward

These are early experiments in a bigger vision: **agents that learn throughout their lifetime**, not just during pretraining or fine-tuning. There’s a lot we still don’t know. How should agents balance diversity versus specificity when storing examples? How do they avoid locking into narrow strategies? Can different agents share self-collected knowledge safely and effectively? How can we rigorously evaluate designs of such stochastically evolving systems?

What’s clear is that self-improvement through self-generated context is a promising step. It turns every solved task into scaffolding for the next, offering a lightweight, scalable path to more adaptive AI systems:

> **Experience shouldn’t vanish once a task is done—it should accumulate, compound, and make the agent smarter over time.**


