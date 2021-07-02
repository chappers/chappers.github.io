---
layout: post
category : 
tags : 
tagline: 
---

Relational data mining is one where the data that is provided isn't a single "flat" table, but rather it is a series of relational data tables. There are several ways to deal with this in data mining research, but in this post we will only cover a really small and specific type - propositionalization. 

In propositionalization, the solution to the problem is simply to convert the relational data to a flat structure. There are several ways of doing this, all with their advantages and disadvantages. 

Suppose that the relationships between entity keys is:

```
A -> B -> C
```

In this setting, we may have many to many relationships and assume that each entity key has a "nice" flat table structure. 

**RollUp**

In the roll up algorithm, the approach is to "roll up" and aggregate the table one layer at a time. In essence the solution would be to reduce the above relationship to 

```
A -> B*
```

Where `B*` now also contains aggregated information that was contained at entity level `C`. From here we will do one more roll up to a flattened table all at the entity key of choice

**RELAGG**

Another solution is the RELAGG algorithm (RELational AGGregationS). In this setting we will change our relationships to a data star schema, that is we will transform our relationships so that they are all one level deep. Then from this setting we will aggregate all information to the single entity level of interest. 

---

To understand the differences and perhaps how these relationships can be created in a graph database in order to generate features/flatten appropriately, I will demonstrate some code using Python's networkx. To do so, I will simulate the relationships above and show diagramatically how the entities would relate using _RollUp_ and _RELAGG_. 

```python
# generate a synthetic example

import networkx as nx

# generate list of entities for a, b, c respectively
a_list = ["a_{}".format(x) for x in range(10)]
b_list = ["b_{}".format(x) for x in range(20)]
c_list = ["c_{}".format(x) for x in range(10)]

# construct set of random relationships
# we have relationship A -> B -> C

import random

ab = []
bc = []
for _ in range(30):
    a = random.choice(a_list)
    b = random.choice(b_list)
    ab.append((a,b))
    
    b = random.choice(b_list)
    c = random.choice(c_list)
    bc.append((b,c))

# construct a graph using networkx
# assuming entity a is our entity of interest. 
# it is trivial to change this relationship - 
# we simply need to change the direction of the relationship
# so that it is always "outward" from the entity of interest

G = nx.DiGraph()
G.add_nodes_from(a_list)
G.add_edges_from(ab)
G.add_edges_from(bc)

```

This will create a directed graph from the entity of interest and then we can easily traverse to create the correct aggregation according to whatever scheme we intend on using. 

For example, consider RollUp relationship:

```python
# select subgraph for a_list[0]
entity_ex = a_list[0]
H = G.subgraph(list(nx.descendants(G, entity_ex)) + [entity_ex])
pos = hierarchy_pos(H, a_list[0]) 
nx.draw(H, pos=pos, with_labels=True, arrows=False)
```

![rollup](/img/rdm/rollup.png)

We can see the hierachical structure which we can traverse naturally to perform whatever aggregation we wish one level at a time. 

Perhaps a simply way is to use RELAGG, as if we have deeply nested relationships it will ignore it, and treat everything with depth of one. 

```python
# RELAGG
entity_ex = a_list[0]
relagg_edge_list = [(entity_ex, x) for x in list(nx.descendants(G, entity_ex))]
H1 = nx.DiGraph()
H1.add_edges_from(relagg_edge_list)
nx.draw(H1, with_labels=True, arrows=False)
```

![relagg](/img/rdm/relagg.png)

