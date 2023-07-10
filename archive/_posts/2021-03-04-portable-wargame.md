---
layout: post
category:
tags:
tagline:
---

As part of reinforcement learning research, I've been thinking more and more about wargames in general, and how they could be codified. Although perhaps not the most interesting, here is some notes on the research I've thought of.

- A portable wargame could be done on a chess board. There are many ways for how this can be used to substitute the usage of a ruler to make unit movement easier (i.e. discrete) - we can imagine these in the form of manhattan distance for example.
- A single square represents one unit of movement, similarly for calculating firing distances
- 45 degrees is calculated based on selecting three adjacent squares of the facing direction; i.e. there are 9 possible directions a unit can face.

```
FFF .FF ..F ... ... ... ... F.. FF.
.X. .XF .XF .XF .XF .X. FX. FX. FX.
... ... ..F .FF .FF FFF FF. F.. ...
```

Where `F` is where the unit `X` is facing, and `.` is the blind spot.

Other variations of this could be the "final fantasy tactics" variation, where flanking from the sides, and behind yield different odds, and a unit can only face in one of the four directions instead, in here the modifies is +1 to roll for the attack if on the side, and +2 if from behind in final fantasy tactics, in this scenario, we may alter it so 1.5 registered effect for flank, and double for rear; thereby lessening the penalty, given the limited coverage in this situation.
