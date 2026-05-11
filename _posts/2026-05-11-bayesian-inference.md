---
title: 'Bayesian Inference'
date: 2026-05-11
permalink: /posts/bayesian-inference
---

Setup and Review 
--- 
This post will serve as a kind of contextual buildup for the next few posts that I plan to write on MCMC and Variational Inference. Without a good understanding about why Bayesian Inference is hard to do in practice, everything else is just a bunch of larp.

Recall Bayes Rule:

$$p(z \mid x) = \frac{p(x \mid z)p(z)}{p(x)}$$

Intuitively, Bayes rule outlines the mechanics to update our belief about some latent variable $z$. We call $p(z \mid x)$ our *posterior* -- literally our belief about $z$ after observing some sequence of data $x$. We compute our posterior by taking the *likelihood*, $p(x \mid z)$ of observing some sequence $x$ given our initial belief or prior, $p(z)$. The denominator is often called the *evidence*. You may think of it as a descriptor about how likely our observation was overall. Without getting too lost in the sauce, I'd like to move to a concrete example to ground these concepts before moving on.

Example 
--- 
Instead of the canonical coin flip example, I'd like to try framing it in a robotics context, specifically a simplified explanation of localization.

The overall question in localization is: given a known map of the world and some sensor readings about my surroundings, how do I know where I am? We can formulate this as a Bayesian Inference where $x$ is some sensor readings and $z$ is my belief about where in the room/world I am. 

At the start, our robot could be anywhere in our room so our prior might be an uniform distribution over the infinitely many points on our map $p(z) \sim Unif$. After receiving some sensor readings $x_i$, we should then attempt to update our belief. 

Recall the parts we need. We have our prior $p(z)$ and need both our *likelihood* and *evidence*. Let's take a look at the likelihood, $p(x \mid z)$ first. The likelihood is a measure of, "how likely are these sensor readings given my prior belief about my position $z$". Notice then, that the numerator of Bayes rule can be seen as a weighted likelihood average across our data samples. *I gloss over the specifics on calculating likelihoods since they aren't relevant towards general conversation about Bayes. See [Chapter 6](https://docs.ufpr.br/~danielsantos/ProbabilisticRobotics.pdf) for more details*

Here comes the hard part, the *evidence*. Computing $p(x)$ is deceptively hard. With our example, let's say each $x_i$ represents some distance to the walls that are in front of our robot. Computing $p(x)$ is then asking "what is the likelihood of $x_i$, given that $p(z) \sim Unif$, I could be ANYWHERE in the map". Since position and angle are continous values, there are infinitely many such positions forcing an integral: 

$$p(x) = \int p(x \mid z) p(z) dz$$ 

Yet even this is a simplification. Even in 2D, poses are at least some x coordinate, y coordinate, and heading angle. So in reality we'd have something closer to, 

$$p(x) = \int\int\int p(\vec{x} \mid \vec{z}) p(\vec{z}) dz_0 dz_1 dz_2$$

In modern robots and especially with deep learning, our latent vectors and observations are much higher dimensions resulting in something largely intractable like:

$$p(x) = \int \dots \int p(\vec{x} \mid \vec{z}) p(\vec{z}) dz_0 \dots dz_{t-1}$$

So Now What?
---
In practice, we almost never compute the evidence directly. Instead, we lean on approximations. In robotics, common approximations are sampling-based methods like **particle filters**, which represent the posterior as a dynamic set of weighted samples, and **Kalman filters** (UKF, EKF), which approximate the posterior with a Gaussian assumption and propagate just its mean and covariance. Both sidestep the integral entirely; Particle filters use sums over discrete samples and Kalman filters use closed-form Gaussian updates.

In the next few posts, I hope to continue to tackle this intractability. Specifically by looking at other sampling based methods Monte Carlo, and variational methods that approximate the true posterior with an optimized tractable approximation. 

As a TLDR, Bayesian inference is how we update beliefs with data. The evidence is hard to compute, so we resort to approximations. That 'hardness' is the foundation for the next few posts.

Questions
---
I keep a list of questions that I had when learning the topic which may be useful for others to think about: 
How can we represent non-unimodal beliefs? Is it ever ideal to spread our belief? 

For sampling, what is a sufficient number of samples to decently approximate our posterior? See [KLD-Sampling](https://proceedings.neurips.cc/paper_files/paper/2001/file/c5b2cebf15b205503560c4e8e6d1ea78-Paper.pdf)

We are sacrificing accuracy for tractability, how 'good' do our approximations need to be?

For localization, how can the topology of our environment limit our confidence in our beliefs (hallway vs asymmetric room)? 

What happens if our prior has 0 probability mass assigned to the true values? Does our Bayes update allow us to recover on its own?

If you know about Markov properties, how can we incorporate our robots actions into our belief updates? What do we need to also know (motion???)
