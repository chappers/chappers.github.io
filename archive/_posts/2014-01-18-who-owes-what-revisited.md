---
layout: post
category: web micro log
tags: [python]
---

Perhaps its just an itch, but I couldn't resist. So here is
[**Wow**](https://gist.github.com/chappers/8412812). As stated before
I thought the best way was to emulate [APScheduler](http://pythonhosted.org/APScheduler/).

Usage is fairly similar. To start Wow simply use.

{% highlight python %}
z = Wow()
{% endhighlight %}

Then to add a payment:

{% highlight python %}
z.add_payment("deposit", 39.25, "Chappers", "Tim")
{% endhighlight %}

Finally to calculate, simply do

{% highlight python %}
print z.calculate_owing()
{% endhighlight %}

Here is the [gist](https://gist.github.com/chappers/8412812).
