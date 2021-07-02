---
layout: post
category : 
tags : 
tagline: 
---

This question has been bugging me, so I thought I'll do a write up for a Python Variation: https://stackoverflow.com/questions/12573512/to-find-all-the-overlapping-date-ranges-from-a-given-list-of-date-ranges

```py
import datetime

# given a list like this
# assume that it is sorted by list[0], otherwise trivial to run
# sorted(date_range, key=lambda x: x[0])
date_range = [
    (datetime.datetime(2000, 1, 1), datetime.datetime(2000, 1, 20)),
    (datetime.datetime(2000, 1, 3), datetime.datetime(2000, 1, 9)),
    (datetime.datetime(2000, 1, 18), datetime.datetime(2000, 1, 23)),
    (datetime.datetime(2000, 2, 1), datetime.datetime(2000, 2, 2)),
]

# return a function which provides true, false, which maps to each element of the list if it is overlapping or not
expected_output = [
    True, 
    True, 
    True,
    False
]
```

Solution:

Its quite easy to provide a really naive solution based on SQL. The approach simply is to observe that this are in the range if: `a.start_time < b.end_time or b.start_time < a.end_time`. This lets us do a self-join to resolve this problem. However, this is a $O(n^2)$ solution which can lead to problems if the list of date ranges is very, very large. 

In order to bring this down the $O(n log(n))$, we can use this approach:

```py
def find_overlap(date_range):
    date_overlap = []
    comp_date = None
    prev_date = None
    updated = False
    for idx, el in enumerate(date_range):
        if comp_date is None:
            comp_date = list(el)
            updated = True
        else:
            if comp_date[0] < el[1] and el[0] < comp_date[1]:
                date_overlap.append(True)
                if updated:
                    date_overlap.append(True)
                    updated = False
                comp_date[0] = min(el[0], comp_date[0])
                comp_date[1] = max(el[1], comp_date[1])
            else:
                comp_date = list(el)
                if updated:
                    date_overlap.append(False)
                    updated = False
                updated = True
    else:
        if updated:
            date_overlap.append(False)
    return date_overlap

date_range = [
    (datetime.datetime(2000, 1, 1), datetime.datetime(2000, 1, 20)),
    (datetime.datetime(2000, 1, 3), datetime.datetime(2000, 1, 9)),
    (datetime.datetime(2000, 1, 18), datetime.datetime(2000, 1, 23)),
    (datetime.datetime(2000, 2, 1), datetime.datetime(2000, 2, 2)),
]

date_range2 = [
    (datetime.datetime(1999, 1, 1), datetime.datetime(2000, 1, 2)),
    (datetime.datetime(2000, 1, 3), datetime.datetime(2000, 1, 9)),
    (datetime.datetime(2000, 1, 7), datetime.datetime(2000, 1, 23)),
    (datetime.datetime(2000, 2, 1), datetime.datetime(2000, 2, 2)),
]

date_range3 = [
    (datetime.datetime(1999, 1, 1), datetime.datetime(2000, 1, 2)),
    (datetime.datetime(2000, 1, 3), datetime.datetime(2000, 1, 9)),
    (datetime.datetime(2000, 1, 7), datetime.datetime(2000, 1, 23)),
    (datetime.datetime(2000, 1, 20), datetime.datetime(2000, 2, 2)),
]

print(find_overlap(date_range))
print(find_overlap(date_range2))
print(find_overlap(date_range3))
```

