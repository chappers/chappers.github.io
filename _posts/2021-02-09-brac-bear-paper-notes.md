---
layout: post
category:
tags:
tagline:
---

In this post, I thought I'll talk through my reading of the [BEAR](https://arxiv.org/abs/1906.00949) and [BRAC](https://arxiv.org/abs/1911.11361) papers in the offline reinforcement learning area.

BRAC and BEAR are related algorithms which make use of the behaviour policy, $\beta$ (i.e. policy that is directly derived from offline data) through modelling the behaviour (sounds redundant but bear with me).

In BEAR, the behavior policy is created through cloning the behavior using an autoencoder $\hat{\beta}$, that is we learn an autoencoder to clone the offline data. If we think about what are the inputs/outputs of $\hat{\beta}$, the answer is we input the state $s$, and we get the output as a distribution of actions $a$, i.e. $\hat{a} \sim \hat{\beta}(s)$.

In BRAC, they offer some variations to this, the notable one is the use of the dual form of divergence (we'll get to this in a second), where the behaviour is instead learned through constructing a discriminator function $g$ to make use of the Fenchel dual (see [Nowozin et al. 2016](https://arxiv.org/abs/1606.00709)). This removes the need to learn the cloned policy at all.

**Taking a Step Back**

Okay, there is a lot to unpack here, and we'll have to go through things one at a time. In BEAR and BRAC, the goal is to "look at Soft Actor Critic" and decide to replace the entropy term which controls the exploration exploitation, with some measure of divergence with the behavior policy $\beta$ (offline data). This constrains the allowed actions to be taken to only actions which have been observed in the offline data.

How do we learn the behaviour policy? (See above) - either through an autoencoder (BEAR), which allows us to retreive the distribution for the state action pair, or learning the discriminator function for the dual form of divergence.

Can we loosen the constraints so that it can explore a little? BEAR suggests using a soft constraint in the form $D(\pi \vert \vert \beta ) - \epsilon$ for some hard constant $\epsilon$, whereas BRAC does away with this altogether (n.b. I know BRAC suggests the primal form is superior but I just wanted to highlight the differences as I found it interesting).

Specifically in BRAC, they propose using the dual form of _reverse_ KL Divergence, so that $f(x) = - \log(x)$ and the conjugate $f^*(t) = -\log(-t) - 1$, which are shown in tables 1 and 2 in [Nowozin et al. 2016](https://arxiv.org/abs/1606.00709).

> An aside, BRAC didn't explictly say they used _reverse_ KL, but its clear when trying to derive that it has to be _reverse_ KL.
