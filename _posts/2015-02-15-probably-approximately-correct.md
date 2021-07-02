---
layout: post
category : web micro log
tags : 
---

There are some things that I don't quite understand, but I thought that writing the high level information about it may help my intuition. Judging from the title it is the idea of being "Probably Approximately Correct" (PAC) in the realm of machine learning.

Let's not worry about the details, but simply look at a few conclusions we can draw:

# For consistent learners...

We can create some equality on the number of training examples \\(m\\), and the allowed error \\(\epsilon\\), with some probability of failure at some level \\(\delta\\), with the size of our Hypothesis \\(H\\), we have the following:

$$ m \ge \frac{1}{\epsilon} (ln |H| + ln (1/\delta))$$

Which we hold for _any consistent learner_ to successfully learn any target concept in \\(H\\).

What do we mean by a consistent learner? Well a consistent learner is a leaning algorithm that outputs a hypothesis that commits zero errors over the training examples. 

But such an example is often unreasonable. What if we relax such an assumption?

# For agnostic learners...

We can create a probability which does this too. It would simply be the following:
 
$$ m \ge \frac{1}{2\epsilon^2} (ln |H| + ln (1/\delta))$$

which provides the agnostic case. 