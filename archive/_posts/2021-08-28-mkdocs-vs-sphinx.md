---
layout: post
title: MkDocs vs Sphinx?
category:
tags:
tagline:
---

So revisiting python packaging and documentation again recently and the only conclusion I have is for a solo developer (or a small team) mkdocs is far superior. You can make do without all the bells and whistles and just get things running.

You'll be forced to write prose rather then autodocs - but that's okay; its probably good practise anyway. Afterall if you consider all the major packages you use today, there is a significant amount of prose in order to help users get started. Infact some relatively large open source packages do not even make it obvious how you would view the lower level API docs (not online at least).

So in summary - if you're getting started, just use mkdocs. Its just so much better...

```sh
pip install mkdocs mkdocs-material mkdocstrings pytkdocs[numpy-style]
```

With mkdocs.yaml

```yaml
theme:
  name: "material"

plugins:
  - search
  - mkdocstrings
```

You'll thank me later...
