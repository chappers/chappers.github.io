---
layout: post
category: web micro log
tags:
---

If you Google documentation with Python, there is a lot of posts and articles on
how you can use sphinx with automatic documentation generation. These work well
however for a simple script `pydoc` is more than enough to the task.

```sh
pydoc script > script.man
```

Or for html files (which seem a bit more ugly):

```sh
pydoc -w script
```

There is certainly an element of elegance to the minimalistic man pages.

As this mimics the `docstrings` used within Python, in some ways it is cleaner
than literate programming approaches such as `Pycco`
