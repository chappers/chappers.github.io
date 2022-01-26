---
layout: post
category:
tags:
tagline:
---

One of the interesting posts to pop up in June 2018 was on [Airbnb and their experiences with React](https://medium.com/airbnb-engineering/react-native-at-airbnb-f95aa460be1c). Whilst I'm obviously not a React developer, there are a lot of golden nuggets to think about in my own work and how I should approach decision making. In this post I thought I'll comment on Airbnb's reflections and what they mean for me, when I'm thinking about taking engineering one step further.

# Airbnb's Goals

What is important when choosing something, is to firstly consider their goals and motivations. It's clear they didn't firstly decide to simply build up their own framework until they could justify doing so (the series of blog post ends talking through their decision to move to their own bespoke framework).

> Our goals with React Native were:
>
> 1.  Allow us to move faster as an organization.
> 2.  Maintain the quality bar set by native.
> 3.  Write product code once for mobile instead of twice.
> 4.  Improve upon the developer experience.

In the same way I believe, that my own personal goals when deciding within an organisation to build our own framework, or to leverage existing ones should have the same flow. In my mind it should read:

1. Allow us to move faster as an organization
2. Train a model code once, deploy everywhere
3. Improve data scientist experience

# Write once, Deploy Everywhere

> The primary benefit of React Native is the fact that code you write runs natively on Android and iOS. Most features that used React Native were able to achieve 95–100% shared code and 0.2% of files were platform-specific (_.android.js/_.ios.js). - [React Native at AirBnB: The Technology](https://medium.com/airbnb-engineering/react-native-at-airbnb-the-technology-dafd0b43838)

> A common misconception is that React Native allows you to move away from writing native code entirely. However, that is not the current state of the world. The native foundation of React Native still rears its head at times. For example, text is rendered slightly differently on each platform, keyboards are handled differently, and Activities are recreated on rotation by default on Android. A high-quality React Native experience requires a careful balance of both worlds. This, paired with the difficulty of having balanced expertise on all three platforms makes shipping a consistently high-quality experience difficult. - [Building a Cross-Platform Mobile Team: Adapting mobile for a world with React Native](https://medium.com/airbnb-engineering/building-a-cross-platform-mobile-team-3e1837b40a88)

These two statements resonate in particular with the goals in Data Science. Some companies, such as Uber and others mandate a single framework for all teams (such as Spark for data engineering and data science as an example, or TensorFlow). This allows teams to easily meet the goals above. In my experience one of the downsides is the need for a _particular_ data engineering and data science team; two teams which are equally strong with one singular framework.

In practise this comes with many organisational challenges, and just like at AirBnB, it is no different. What happens if your data engineering team in Spark works in Scala? And your Data Scientists work in Python and R? What if they write code specific to Python and R and wish to deploy (user defined functions)? What are the downsides there?

## On Apache Spark

Apache Spark is an amazing framework. Being able to scale out analytics has never really been easier. However, in my experience, no matter how you write your Python code, just like the quotes above, it will eventually rear its ugly head; you are going to have to read Scala code to understand and debug correctly. Worse still if you throw R into the mix with libraries like `sparklyr` running on top; now your software engineers will have to understand R syntax in order to assist in handling debugging issues in production.

The solution may be indeed to learn from how AirBnB approached this and their resolution:

1.  Identify where the strengths of the chosen framework is and leverage it. In Spark for me, we have UDFs and HiveQL. Writing code in this fashion ensures that everyone can easily reason with the code as long as they know SQL.
2.  Identify potential weekends and understand that it can solve all problems. In the situation above, UDFs would be the issue. Ensure that you have approaches to correctly mitigate or provide an exit strategy for those situations. This may mean coding particular UDFs in Scala and calling them in Python (this is certainly possible) or ensuring that there are clean, distinct interfaces in how to interoperate with different frameworks

Personally despite their speed (or rather lack thereof), I believe UDFs are extremely powerful. If done correctly or cleverly, we can be in a situation where a Python user can code a UDF that works in both Pandas and Spark - and able to test both frameworks at once with little effort through the use of decorators. This touches on how planning ahead and encouraging particular patterns may _improve developer happiness_.

# Having an Exit Strategy

Knowing how to quit and understanding when you've outgrown patterns is extremely important. For AirBnB it was when:

- The cost of writing code once to be deployed twice exceeded the cost of writing and maintaining two sets of code
- It no longer allowed people to move fast

However, what is important to learn or understand is that AirBnB simply outgrew those patterns. Which then leads to how do we sunset these things?

> An alternative route is to gradually create a new system around the edges of the old, letting it grow slowly over several years until the old system is strangled. Doing this sounds hard, but increasingly I think it's one of those things that isn't tried enough. - [Strangler Application](https://www.martinfowler.com/bliki/StranglerApplication.html)

Maybe this is something to think about, but is relevant for a post at another time.
