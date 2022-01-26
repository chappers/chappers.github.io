---
layout: post
category:
tags:
tagline:
---

A couple interesting blog posts have been making the rounds on [VendorOps](https://rachelbythebay.com/w/2020/08/14/jobs/) and also [Google's notional idea of deprecation](https://medium.com/@steve.yegge/dear-google-cloud-your-deprecation-policy-is-killing-you-ee7525dc05dc) have been making the rounds, and it got me thinking about my own role and career; how is this being reflected in the ML community?

I think it has been quite invasive! The rise of "building things" and "breaking things" for ML research is still quite common (probably given that its more a research driven domain than an engineering domain right now), and the solution to these problems is simply to "throw things onto docker". By natural extension, vendor-lockin becomes a real thing to contend with, because hosting these kind of infrastructure is highly non-trivial (in general).

What I think we as an industry need to reassess is the whole machine learning engineering and production toolchain - why are we relying on programming languages like Python which has severe backwards compatibility issues over another language or ecosystem which is more "long-term" focussed? Do people actually expect the software we write to exist in the long term? From a corporate standpoint; these things don't _really_ matter - however I'd argue from an ethical software engineering standpoint we need to rethink how algorithms are used and productionised into systems.

Right now, I stare down the roadmap and perceive that we're going down the wrong direction

- software is tied to hardware (looking at you nVidia and Deep Learning community)
- software is not designed to run "forever" or "offline" in the 90s sense - rather its designed to be written by and supported increasingly by vendors.

From an open-source software perspective, I think we can do better. From a corporate perspective, these things make good financial sense.
