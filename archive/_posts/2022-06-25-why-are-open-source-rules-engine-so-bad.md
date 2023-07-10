---
layout: post
category:
tags:
tagline:
---

Honestly, finding a rules engine sucks. Most of them "work" for simple web-based applications for determining permissions, but none are anywhere good enough to replicate an expert system. 

There are probably many, many reasons why this is the case:
- Open source excels are _libraries_ not _applications_. I don't recall the last time I actively supported an open source _application_.
- Creating a one-size-fit-all for expert systems is hard work! I want a decision tree-esque set up that is configuration driven. There aren't many options for that. 

So I'm going to resort to writing my own. It didn't end up being too difficult, but the user interface/interaction isn't great and will have to rely on other tools and workflows for deployment. This probably reflects why you would never see something like this in the wide in an open source library...

Rough idea:
* Create rules which acts as nodes in a decision tree
* Link the nodes by implementing via a binary tree structure
* Visualise the nodes using Mermaid

Then we can easily evaluate the binary tree using the rules and visualise it. The next challenge would be how do we integrate it with workflows so that we can deploy. I'll leave that for later, though the rough idea is to use existing CICD tooling and workflows to enable this such as integration with GitHub actions etc. 

This notion also aligns with Martin Fowler's idea of creating a small tightly controlled DSL for a specific usecase, rather than trying to create an all encompassing system.