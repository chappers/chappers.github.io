---
layout: post
category : 
tags : 
tagline: 
---

I've been reflecting on how games simulate real-time events and how reinforcement can be used to leverage that. In FTL, a game which has "real-time" combat, it does it in a mostly automated way:

*  Moves are more or less resolved by "ticks" which we can say occur at 1 second intervals
*  Attack Moves are boxed into either 2x1 or 2x2 configuration (thereby reducing the complexity of the space) - that is only agents in the same room can attack each other, and there is no agent specific targetting required; it is all resolved heuristically
*  The movement which an agent can make are mostly "linear" in nature (thereby no path finding required) - that is it is restricted by the design of a "ship", so that it isn't really an open world.

To expand this; what might this mean from a game design or AI design perspective?

It suggests that we effectively reduce the amount of choices one has to make - but can still give rise to a an approach which requires deep macro (resource management) and micro (timing of battles) approach. It is in these realms which reflect real-time gaming, but is entirely decision based which makes it exciting - rather than a reflect or skill based approach like building AIs in Starcraft. 

To that end, what I would like to see is a:

*  More open world which consists of these decisions (units resolve fights automatically in real-time, but only high order commands like capture an area can be provided)
*  higher-order position given precendence over miromanagement of units/decisions (how would you make this fun?)
*  Longer term planning embedded as part of the design.

Things like "Total War" are more similar to the real-time resolution which I would be interested in (fights resolved automatically in real-time), with base building elements to manage resourcing during battle. 

