---
layout: post
category:
tags:
tagline:
---

One useful pattern in machine learning is having time-based constraints for testing and comparing your algorithms - afterall that is the easiest way to assess its efficacy. The easiest way to do this is through the [`stopit` library](https://github.com/glenfant/stopit).

The simpliest pattern I got working is this one:

```py
from stopit import threading_timeoutable as timeoutable
import time

class SimpleObject(object):
    """
    >>> ss = SimpleObject()
    >>> ss.ticker(timeout=5)
    >>> ss.items
    """
    def __init__(self):
        self.items = []

    @timeoutable()
    def ticker(self):
        while True:
            self.items.append(time.time())
            time.sleep(1.0)

```

What are some of the implications or usage? Within my work, we can "abuse" this function in many ways:

```py
def ticker():
    # do stuff
    # pretend this does cool ML stuff for one iteration (e.g. one model eval)
    time.sleep(1.0)
    return time.time()


class PerformanceTracker(object):
    """
    Usage:

    >>> pt = PerformanceTracker()
    >>> pt.tracker(timeout=5)
    """
    def __init__(self):
        self.items = []

    @timeoutable()
    def tracker(self):
        while True:
            self.items.append(ticker())

```

This could be a useful pattern for many crawlers as well! Surprisingly, there are very few examples using this library on the internet...
