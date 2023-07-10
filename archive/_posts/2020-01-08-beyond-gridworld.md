---
layout: post
category:
tags:
tagline:
---

Gridworld is a commonly used environment for Reinforcement Learning (RL) tasks; it is simple to implement and understand for the purposes of evaluating a wide number of different agents. In a nutshell Gridworld presents:

- A grid-based world, which can be easily represented as a matrix (or by extension a tensor, if its a world with different 'layers')
- A simple turn-based mechanism for which agents interact with the world
- A set of rules and actions which are evaluated per 'step' within the RL environment.

However, grid worlds present several deficiencies in common implementations of real-time games, these include:

- Understanding a 'tick' in the context of game development, when all actions are realised in the game loop
- Understanding the time it takes for an action to be realised (e.g. turning rates in games, animation cancelling, projectiles being disjoint)
- Solving or dealing with conflicts or discrepancies in core game logic

Within AI research, we don't really talk about how these are dealt; instead they're typically embedded as part of the game engine which is used (after all the RL agent only reacts to the environment). There are important nuances to consider in these contexts with respect to understanding bugs and why some environments or strategies can't be "moved over" from one environment to another!

## microRTS

microRTS is an interesting environment whereby conflicts or discrepancies cause agents to resolve to the "last known legal state". As actions such as attack or move may require units to be adjacent to each other in _real-time_ this can cause "ghosting" issue whereby melee units cannot "reach" each other.

```
a --> <-- b
```

> If unit `a` and `b` both continually attempt to reach the same cell, they will continually remain in the same position; as the game engine simply reverts tot he last known state.

In most typically 2D RL environments, these aren't really an issue, as each agent takes one action after another (e.g. the player controlled agent makes a move, and the opposing player responds, like in chess, tic-tac-toe, go, hanabi etc)

This begs the question how do "real" RTS games solve this issue?

## Starcraft

Starcraft and by extension Warcraft, Dota 2 and more simply resolve this issue, by having unit sizes and attack ranges. Melee units successfully attack units which are in melee range. Melee range do not need to be directly adjacent to each other, and there is an "allowable" gap between them. In the original Starcraft, units were approximated by a $8 \times 8$ grid, so that collision detect and more was handed not using a single cell, but a group of cells. Although this is obvious, this has implications on how we construct environments for the purposes of RL (as well as complexity in representation and the state information which we feed the agents).

What this means is that units can now occupy larger spaces, and "look like" they are adjacent and resolve it efficiently. The larger the unit the larger the permissible gap which we can have between the units.

## What's Next?

For me, the important thing is to think through how we can apply these ideas to RL environments to have better simulation of game playing systems which can be easily replicated and built within a variety of different environments. Even on first impressions they are highly non-trivial, however I do believe in the future, we can come up with "cheaper" constructs to simulate multi-agent environments in real-time which interact with heterogenous agents and other custom constructs in a fluid manner.

**Update**: taking a look at "Combat" mode in Advance Wars series demonstrates how they transition from turn based to an experimental "real-time" mode. It consists of moving units within a $4 \times 4$ grid which allows for "smaller" movements and collision detection. All attacks are treated as "ranged" attacks for the purposes for determining whether units are adjacent or not. Since advance wars was traditionally a turn based game; this provides a glimpse of what was thought to be appropriate!
