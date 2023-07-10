---
layout: post
category: web micro log
tags:
---

Lately I've been thinking about software design. What is considered "good" design?

As a meta question, [does design even matter](http://www.posteriorscience.net/?p=206)?

## The Unix Way (Keep it simple stupid!)

There is a lot of different views of [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy), and many people have tried to summarise it. The general feeling is that unix favours:

1.  Simple: make each program do one thing and one thing well.
2.  Modular: output of a program should be an input to another, unknown, program. As such avoid binary input formats, and favour portability over efficiency

There are lots of variation to this, and what I gather is that the Unix way attempts to break down the problem in such a way that each part can handle it in the most sensible way possible.

## Rob Pike's 5 Rules

Rob Pike is undoubtedly another person who has both influenced and influenced by ideas shaped in Unix. His [five rules](http://users.ece.utexas.edu/~adnan/pike.html) are repeated below

- Rule 1. You can't tell where a program is going to spend its time. Bottlenecks occur in surprising places, so don't try to second guess and put in a speed hack until you've proven that's where the bottleneck is.
- Rule 2. Measure. Don't tune for speed until you've measured, and even then don't unless one part of the code overwhelms the rest.
- Rule 3. Fancy algorithms are slow when n is small, and n is usually small. Fancy algorithms have big constants. Until you know that n is frequently going to be big, don't get fancy. (Even if n does get big, use Rule 2 first.)
- Rule 4. Fancy algorithms are buggier than simple ones, and they're much harder to implement. Use simple algorithms as well as simple data structures.
- Rule 5. Data dominates. If you've chosen the right data structures and organized things well, the algorithms will almost always be self-evident. Data structures, not algorithms, are central to programming.

The first two rules are derived from "premature optimization is the root of all evil", whilst 3 and 4 are reminiscent of the KISS principle above.

The last one is more interesting, coming from Fred Brooks. Fred Brooks has a series of essays on software development. Although it is fairly old, various principles still apply. He is of course famous for the essay "No Silver Bullet", that there is not one way which will solve all software design problems. Perhaps a less famous one is the notation "Representation Is the Essence of Programming" - or in other words "write stupid code that uses smart objects".

## Last Thoughts

Like many things we have to think about all these comments and principles in context. For me, my current thought process is around use of frameworks and ensuring flexibility and portability.

Questions that I have:

- Programming via (global) configuration; should we do this?
- How do you build small programs from an analytics standpoint?
- What about SOLID principles and software architecture which isn't even touched on or considered here?
- What are the consequences of integrating original research into analytics pipeline?
- What about Design Patterns? [Peter Norvig has commented that OOP design patterns become irrelevant when moving to dynamic languages](http://norvig.com/design-patterns/).

### What Else Should I read?

- [The Architecture of Open Source Applications](http://www.aosabook.org/en/)
- [Machine Learning: The High-Interest Credit Card of Technical Debt](https://research.google.com/pubs/pub43146.html)
