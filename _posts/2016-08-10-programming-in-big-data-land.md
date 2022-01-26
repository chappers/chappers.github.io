---
layout: post
category: web micro log
tags:
---

_This post is more of a stream of consciousness about big data technology and the things I do everyday_

Big data is great; we all talk about it, and when you run fancy tutorials there are nicely configured virtual machines with point and click (also with GUI's!) to allow you to get aquainted with big data.

However on the practical side, companies don't often have nice vendor solutions for you, and often you find yourself staring at a blank terminal screen...

Welcome to big data!

Life isn't always full of nice IDEs as Jupyter, or RStudio (I found Jupyter to be a huge hassle to get working with Spark). Maybe you're fortunate to have a nicely configured Zeppellin instance for you to work off. If you have all these tools and they've been configured well...then great! You don't need to read this.

## Enter the Console

As an example, of what working in big data feels like for me, try installing `vagrant` and `virtualbox` and try out one of the spark boxes:

```
vagrant init paulovn/spark-base64; vagrant up
```

Now you can `ssh` into this:

```
vagrant ssh
```

For most developers this isn't a problem, but for someone who grew up on GUI interfaces this is a massive problem and learning how to navigate this realm is probably the first thing you should learn!

## Playing with Spark

Without a doubt the best way to learn is simply by keying things in the console. If you came via Vagrant; I would recommend you learn how to connect PuTTy or WinSCP or similar tools in order to copy files in and have multiple sessions; one screen to run the code and another screen with `vim` open. This is the usual pattern.

The best way to begin is from the beginning; using:

```
spark-shell
```

or

```
pyspark
```

Welcome to REPL.

## Closing Thoughts

There are many ways to make your life easier. For example, installing Jupyter and using Jupyter notebooks (remember to port forward) will give you something...more interactive!

My favourite way is to run Sparkling Water which can then be access from your local machine (remember to port forward).

After that you should be on your way to learning about big data.

Set your expectations low, and enjoy yourself. No one said it was easy.
