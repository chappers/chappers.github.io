---
layout: post
category: web micro log
tags:
---

[Ostinato](https://github.com/chappers/ostinato) is my attempt at writing an ostinato for choral music. At the time of writing this blog post, I must say it was a lot harder than I anticipated for the following reasons:

- Limited support of any modules for music editing
- Personally, limited knowledge and patience for the existing modules!

I used the [abjad](http://abjad.mbrsi.org/) module, which although contained documentation, it quite frankly might as well not be there. I still haven't figured out how to extract the key or time signature from the `LilypondParser()` class, and resorted to a regex hack since I was just spending way too much time on a problem which should have been quite trivial.

Overall it has been a learning experience, learning a bit of [LilyPond](http://www.lilypond.org/) and having a crack at this problem.

Currently it could be improved in the following ways:

- Determine where the start of a bar is, and consequently where each beat falls!
- Have a recursive method to determine the optimal chordal progression based on music theory

But enough with the ranting! What does this code actually do?

Here is "Mary had a Little Lamb" written in LilyPond:

```
\relative c'
{
\time 4/4
e d c d
e e e2
d4 d d2
e4 g g2
e4 d c d
e e e e
d d e d
c1
}
```

Which will generate the following output:

![Mary had a little lamb](/img/ostinato/marylamb.png)

Running the code in the repo above would generate the below:

![Mary had a little lamb](/img/ostinato/marylamb-lh.png)

The idea behind the code is simply iterate through every single note and replace it with the appropriate chord. However in the future, you would want to alter the frequency that the chord appears, (e.g. if it was 4/4 time only on the 1st and 3rd beat), or have variations such as using arpeggios, or various other musical devices. This idea could be also extended through somehow inferring chordal progression in the piece of music.

There could be much room for future development!
