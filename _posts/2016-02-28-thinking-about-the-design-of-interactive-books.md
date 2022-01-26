---
layout: post
category: web micro log
tags:
---

Recently I've been thinking about the design of interactive books. If you check out ideas shown by [Bret Victor](http://worrydream.com/), you can see
some of the ideas he has showcased with regards to interactive documents. I think the challenges of
creating interactive books stem from not only on the technical challenge, but making them interesting even when they are in a
hardcopy format.

I think the newspaper to digital news medium is one interesting medium which is worth considering and comparing.

One aspect of the rise of gifs and tableau dashboards is that these articles still make sense and are
beautiful despite the lack of interactivity. Unfortunately when [performing code exercises](http://worrydream.com/#!/LearnableProgramming)
the interface isn't particularly beautiful or useful if it wasn't interactive. Now it is true that this interface
is still going to be useless when it is in hardcopy form (the coding is useless except for rote learning
which Bret Victor is trying to avoid); it may still be useful when teaching key concepts in statistics
and mathematics.

Infact the lack of "standard" way of programming mathematics is probably one of the biggest barriers
to creating interactive books for the education of abstract concepts. I would argue that there are
no useful ways to even have an "offline" version of an interactive document as the only one which I can think of is
the usage of Jupyter notebooks for offline and online viewing.

Perhaps this could be a project to think about in the future.

n.b. I am aware that d3 exists, unfortunately d3 violates the ability to easily program mathematics into
its documents, unless the backend is run on a server. For example, imagine creating
an interactive diagram for the various decision boundaries for the K-nn model.
