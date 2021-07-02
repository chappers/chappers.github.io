---
layout: post
category : web micro log
tags :
---

**Teaching Machines to Think** is a series which I am intending on starting. The focus on the series is on the engineering considerations made when we train machine learning and artificial intelligence algorithms.

<iframe width="560" height="315" src="https://www.youtube.com/embed/j4tVluwCIlQ" frameborder="0" allowfullscreen></iframe>

We can create a Naive Linear Regression Solver:  
1. Guess what `m`, `b`  
2. Make another guess on the incremental change for the best observed `m` and `b`  
3. Compare the proposed improved with our best `m` and `b`. If it is better then we take the proposal, if it is worse then we leave the proposed `m` and `b` alone.   
4. Repeat 2 and 3 till we're happy.   

In this approach it raises a few questions  

*  Statistic question: What is the computational cost of closed form over this approach?  
*  Optimization:  
    *  RHC can we do better? What is the ideal step size  
    *  Similated annealing? What is the penalty over time that we should have for our steps?  
    *  Covex optimization? SGD  

---

As starting this series I have decided to grab a new microphone; the Rode NTUSB Microphone. This microphone is quite amazing and fairly priced. It has definitely has many features over my webcam microphone. It is very easy and straight-forward to use.
