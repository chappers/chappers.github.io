---
layout: post
category: web micro log
tags: [python]
---

As a farewell gift from my colleagues at Westpac, I was given a [Fibit Flex](http://www.fitbit.com/au/flex)! Lucky me!

One of the cool things about Fitbit is the dashboard which has a wealth of information in it. But for me, its not enough.

![my-dashboard](https://raw2.github.com/chappers/chappers.github.com/master/img/fitbit-data/my-dashboard.png)

I don't want 15 minute segments, I want it at a finer scale! Also, what if I don't really care about steps taken, but rather I want to know distance travelled. Luckily the Fitbit API allows you to download data at a finer scale and extract distance travelled information, though personally I would like it if the data availabe was even finer.

Below is a plot showing distance travelled in 1 minute segments from 6-8pm during my social futsal game yesterday night.

![2013-11-02-futsal](https://raw2.github.com/chappers/chappers.github.com/master/img/fitbit-data/2013-11-02-futsal.png)

The data is indeed quite messy, and perhaps too noisy to have pretty visualisation compared with the corresponding dashboard picture. But having the option to custom tune what you want is important to me.

Furthermore, the Fitbit dashboard didn't allow me to zoom in. With so many lines on the screen it makes it difficult to visualize only my Futsal activity.

[ipython notebook](http://nbviewer.ipython.org/7286772)  
[source](https://gist.github.com/chappers/7286772)
