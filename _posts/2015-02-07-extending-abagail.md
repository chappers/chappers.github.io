---
layout: post
category : web micro log
tags : 
---

I'm currently working through my second assignment for CS7641 at Georgia Tech and I thought I'll record some of my personal notes and coments on the [ABAGAIL](https://github.com/pushkar/ABAGAIL) library. Since I'm not primarily a Java developer, working of this code obviously took longer than a seasoned Java developer. As such I wouldn't be surprised if the methods and approaches I used here are:

*  Not efficient  
*  Outright wrong!  

Nevertheless, here are my attempts to extend the functionality in ABAGAIL. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/oFvQsArCSXo" frameborder="0" allowfullscreen></iframe>

# The Setup

The first thing I tried (and failed) was to see if there was a good Jython IDE editor. There isn't one that I found that was easy enough to use. I ended up just coding in notepad which was infact more pleasant than trying to dive through confusing IDE settings. 

With the Java IDE I settled on IntelliJ Community edition, simply because I have used Android studio before and they are quite similar. Building the `*.jar` files for ABAGAIL were rather trivial and I had no hiccups there. However by now I had already (unfortunately) wasted three nights fiddling around with settings and playing around with the sample `jython` code which was provided. In terms of actually making progress on the assignment, I can say I made none. 

# Implementing my own `EvaluationFunction`

There isn't much fun just using what was given in the examples. It is far more exciting and rewarding implementing something that you created yourself. So I began with the rather trivial example, maximising a parabola of the form:

$$ y = -(x-m)^2 + b $$

In order to accomplish this, we would quite naturally have to write some Java. Mirroring the other functionality in the `opt.example`, we can quite readily create such an `EvaluationFunction`.

```java
public class ParabolaEvaluationFunction implements EvaluationFunction {

    private double xAxis;
    private double intercept;

    public ParabolaEvaluationFunction(double m, double b){
        xAxis = m;
        intercept = b;
    }

    public double value(Instance d) {
        Vector data = d.getData();
        return (-((data.get(0)-xAxis)*(data.get(0)-xAxis))) + intercept;

    }
}
```

Although it felt extremely difficult before, most of it was getting my head around what exactly `Instance` did. But other than that it was quite simple to implement. The next part was a little more tricky, since I (still) don't completely understand what the `Jython` code was doing in the setup of the reinforcement learning file. But nevertheless following the same vein as the examples, we can quite quickly and easily create the correct script:

```py
m = 6
b = 7
N=1
fill = [100] * N
ranges = array('i', fill)

ef = ParabolaEvaluationFunction(m, b)

odd = DiscreteUniformDistribution(ranges)
nf = DiscreteChangeOneNeighbor(ranges)
mf = DiscreteChangeOneMutation(ranges)
cf = SingleCrossOver()
df = DiscreteDependencyTree(.1, ranges)
hcp = GenericHillClimbingProblem(ef, odd, nf)
gap = GenericGeneticAlgorithmProblem(ef, odd, mf, cf)
pop = GenericProbabilisticOptimizationProblem(ef, odd, df)

rhc = RandomizedHillClimbing(hcp)
fit = FixedIterationTrainer(rhc, 200000)
fitness = fit.train()
print "RHC fitness : ", fitness
print "RHC optimal : ", rhc.getOptimal()
print "RHC: " + str(ef.value(rhc.getOptimal()))

sa = SimulatedAnnealing(1E11, .95, hcp)
fit = FixedIterationTrainer(sa, 200000)
fitness = fit.train()
print "SA fitness : ", fitness
print "SA optimal : ", sa.getOptimal()
print "SA: " + str(ef.value(sa.getOptimal()))

ga = StandardGeneticAlgorithm(200, 100, 10, gap)
fit = FixedIterationTrainer(ga, 1000)
fitness = fit.train()
print "GA fitness : ", fitness
print "GA optimal : ", ga.getOptimal()
print "GA: " + str(ef.value(ga.getOptimal()))

mimic = MIMIC(200, 20, pop)
fit = FixedIterationTrainer(mimic, 1000)
fitness = fit.train()
print "MIMIC fitness : ", fitness
print "MIMIC optimal : ", mimic.getOptimal()
print "MIMIC: " + str(ef.value(mimic.getOptimal()))

```

Which unsurprisingly comes to the right solution:


```
RHC fitness :  6.996995
RHC optimal :  6.000000
RHC: 7.0
SA fitness :  3.183305
SA optimal :  6.000000
SA: 7.0
GA fitness :  -8001.420405
GA optimal :  8.000000
GA: 3.0
MIMIC fitness :  6.983
MIMIC optimal :  6.000000
MIMIC: 7.0
``` 

Hopefully its an interesting take on how you can combine Python and Java in a machine learning setting.


