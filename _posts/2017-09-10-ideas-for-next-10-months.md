---
layout: post
category:
tags:
tagline:
---

Planning for 10 months is always going to be hard, but sometimes it is worthwhile to note it down so that you can review later to see what has changed and arrange any crazy ideas that you might have.

## Research

Within the next 6 months, one achievable goal is to complete a paper which is "good enough" to think about publication. The goal isn't necessarily to publish, but to have something which we can at least make a decision whether it is sufficient down the road. In fact, I would like to push this idea to the next 2 months, where the time is highly dependent on actually producing a framework and repeatable experiments.

**Idea**

Determinantal Point Process (DPP) for diverse feature selection on streaming data. The high level goal is to ensure that new incoming features are sufficiently different as to promote exploring new feature sets which may contribute to the machine learning/data mining problem. This is done in two parts:

1.  DPP selection algorithm. Which presumes no streaming
2.  Conditional DPP, which selects streaming data conditional on the current feature set

Initial results are promising under certain conditions.

Other applications for this could be using DPP to encourage diverse model sets for ensemble/stacking.

**Blockers and Challengers**

One of the biggest blockers is having worthwhile challengers to this problem. There are a couple candidates which I can think of:

- Grafting
- Online Group Feature Selection
- Alpha-investing

**Frameworks to Build**

In order to have a reusable process for streaming data, my current plan is to build a scikit-learn framework (using pipeline idea). Broadly speaking it will be composed of components which look like this:

- "base pipeline"
- "incoming stream 1"
- "incoming stream 2"

and so on. Then the pipeline can be composed as follows:

```python
FeatureSplitter(
	FeatureUnion(
		[('base', BasePipeline),
		 ('stream1', StreamPipeline),
		 ('stream2', StreamPipeline)
		]
	)
)
```

Where `FeatureSplitter` will be a custom class to split off a stream of features by name to ensure that streaming features do not trip anything up.

### Broad Plan and Milestones

For this piece of work the upcoming milestones which I will keep track of are:

- Create framework (1-2 weeks)
- Port grafting algorithm, OGFS, alpha-investing (2 weeks)
- Run experiments on synthetic datasets (1-2 week)
- Write draft paper (2-4 weeks)
- Re-run experiments on ML benchmark datasets/replicating various papers (3 weeks)

Possible parts where I will trip up:

- Framework might be hard to code up well and might have to be scrapped and re-built several times
- May underestimate the time required to do grafting in a "nice" setup. OGFS and alpha-investing have draft layouts created already and should be easier. Worse case scenario, I may remove alpha-investing approach, as that is somewhat dependent on the criterion used to assess incoming features. With multiple criterion and multiple choices of statistical tests, it may become very difficult to experiment and might be sensible to leave to later.

### Future Direction

Beyond DPP, the focus I would like to have is on grafting and gradient descent approaches. I believe that extending ideas to non-linear class of models and especially tree base models have large implications.

Some ideas include:

- extending VFDT algorithm to boosting variant, using grafting ideas in an stacking sense
- Making use of RuleFit algorithm to graft future features, and then reconstructing decision trees afterwards

## Fitness and Health

Health has always taken a backseat when study gets tough. Which means I will put something in here about it. Health comes in two parts:

- Diet
- Exercise

#### Diet

Without a "steady" or proper diet plan, it may come to a time when I should perform some kind of meal prep. What this looks like is a bit unknown at this stage. Portion control and not overeating are probably two parts which are the most important.

The goal is in the long run to loose a bit of weight with the aim to maintain what I have currently rather than making drastic changes.

#### Exercise

Exercise routine will probably consist of the ["benchmark" crossfit workouts](https://wodwell.com/wods/). In a weird kind of way these are quick and push my limits at least.

Workouts completed:

**Fran**

```
15-12-9 (should be 21-15-9)
Thrusters (40kg) + pullups
```

**Jackie**

```
Row 1000m, 50 Thrusters (20kg), 30 pullups
```

**Helen**

```
3 rounds
run 400m, 21 KB swings (24kg), 12 pullups
```

Future workouts:

**Elizabeth**

```
21-15-9
cleans (60kg), ring dip
```

**Grace**

```
30 clean and jerks
```

**Bear Complex**

```
7 unbroken sets for weight
power clean, front squat, push press, back squat, push press
```

**DT**

```
5 rounds
12 deadlifts, 9 hang power clean, 6 push press/jerk (all @70kg)
```

... and probably many more! There are deficiencies in my movement probably around things like:

- Handstand
- Dips
- Muscle ups

but I guess I will have to address them slowly.
