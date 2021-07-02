---
layout: post
category : web micro log
tags : [opinion, data mining]
---



The biggest lesson I've learnt in my short time in analytics is being vigilant on sticking with a _single_ _source_ _of_ _truth_.

Every extract should be run once, cleansed through once, and then thrown into production.

There should not be multiple extracts for the same 'theme' as this serves only to confuse not only other people but yourself (at least in my experience).

Simplicity should be strived for; having one go to source will solve that issue.

I guess the key thing we want to remember is that machines are infallible, but humans are. We will forget, we will make mistakes. Which is why its even more important to stick with one single source of truth. That way we can't forget, we will reduce mistakes. That is the important thing to remember.

##Extract to be Run with Error?

Of course data extracts should work the first time around! That in itself is a no brainer. But what about ever changing database and ever changing requirements due to addition of new products and projects?

Although I'm not thoroughly convinced of the agile philosophy, the idea that the product should be iteratively improved resonates heavily here. Software/code in this environment should never be considered 'finished' but rather in a continual state of improvement; in collaboration with stakeholders. This will mean that the code should be trimmed constantly rather than expanded.

Of course code should run without errors, but not at the expense of making overly broad assumptions or vague statements for the sake of 'correctness'.


