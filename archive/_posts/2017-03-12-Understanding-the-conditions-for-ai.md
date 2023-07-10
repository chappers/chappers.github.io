---
layout: post
category: web micro log
tags:
---

We hear about the amazing things of various AI software, for example Watson. It would appear based on first glance that things like Watson will replace humans very quickly, but how exactly are the metrics for Watson or other frameworks measured?

One of the biggest and most important algorithms are centered around correlation. These are extremely powerful models which pop up in many situations, such as recommender systems, or how medical portion of Watson works; whereby it "intelligently" (via determining correlations) see the interaction among different kind of medicines or diagnosis which describes a patient state. Since it has such a large database of up-to-date knowledge it would probably determine the correct course of action far better than a doctor who is relying purely on their memory.

This leds to:

## Conditions for AI

At this point in writing and thinking about AI, I think there are two conditions for AI:

1.  Replacing human defined "rules" for better rules (for example, the medicine example above)
2.  Perception problems, where humans can already "solve" the problem, but has much difficulty expressing the fuzziness to a machine to solve the same problem (for example image recognition).

In the first case, machines can only learn off data which human can produce.

For example, the current framework of how Watson would work in a medical setting would limit it to merely "learning" off a database of human created (and probably curated) journals. It is not capable in thinking or determining new approaches to improve care or find a cure using an innovative approach which hasn't been created before.

In the second case, often these tasks can already be solved by a human, but rule based approaches make it difficult to solve for a machine. This is commonly associated in image classification. Take for example trying to teach a machine (or an alien if it is easier) to recognise a cat or a dog:

- Cats are smaller (there are cats bigger than the smaller dogs though - also in the context of a photo, how would we know that the animal is "small" or "big"?)
- Cats have triangular ears, whilst dogs do not (there are certainly dogs with cat-like ears)
- Cats have claws, dogs have paws (how exactly would a machine know the difference here?)

And the list can go on.

In this case, deep learning is focused on creating "feature vectors" in order to determine and classify cats and dogs correctly.

## Conditions for Failure in AI Projects

In my (limited) experience, perhaps one of the greatest failure in AI projects is the inability to identify which of the two problems above, you're trying to solve. This leads to a rather vague scope and sets the project up for failure.

Instead what needs to be addressed is simply:

1.  Which scenario above are we trying to solve? Something which human can define (which is in the realm of robotic processing)
2.  Or, is it something which is difficult to define, and probably requires specialised knowledge.

Once we have this understanding we will also need to rephrase this with respect to stackholders expectations. One way to think about this is:

1.  In the first situation above, robotic processing will lead to rules and hence improve _explainability_ of our AI system
2.  In the second situation, we probably won't understand the conditions of which an AI solves a problem

In practise the two lines above are obviously quite blurred, but they are neat guidelines to determine what are the requirements for an AI project.
