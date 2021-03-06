---
layout: post
title: thoughtlog
---

[Simplying board game concepts idea](https://www.jefftk.com/p/simplifying-board-games) - might be useful for RL

For making documents looked scanned:

```
convert -density 90 input.pdf -rotate 0.5 -attenuate 0.2 +noise Multiplicative -colorspace Gray output.pdf
```

```
https://python-prompt-toolkit.readthedocs.io/en/master/pages/full_screen_apps.html when I have some time to write full page appications!
```

```
https://minnenratta.wordpress.com/2017/01/25/things-i-have-learnt-as-the-software-engineering-lead-of-a-multinational/
```

---

When we consider journal and conference rankings what do we use?

For journals we can use the ranking based on the Thomson Reuters Journal Citation Reports, for Conference papers, we can use ERA (Excellence in Research in Australia) or Qualis, which is searchable here: www.conferenceranks.com

---

[Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)

---

Installing CNTK with Nvidia 780 (GPU)

https://docs.microsoft.com/en-us/cognitive-toolkit/Using-CNTK-with-Keras

---

**PredictionIO install script**

```
https://raw.githubusercontent.com/apache/incubator-predictionio/release/0.10.0/bin/install.sh
```

Something like

```
chmod +x install.sh
./install.sh -y # or something like that.
```

---

[Talk of tech innovation is bullsh\*t. Shut up and get the work done](http://www.theregister.co.uk/2017/02/15/think_different_shut_up_and_work_harder_says_linus_torvalds/)

> "It's almost boring how well our process works," Torvalds said. "All the really stressful times for me have been about process. They haven't been about code. When code doesn't work, that can actually be exciting ... Process problems are a pain in the ass. You never, ever want to have process problems ... That's when people start getting really angry at each other."

---

[PyPi Package submission](http://peterdowns.com/posts/first-time-with-pypi.html)

Basic Instructions is to (assuming everything is ready to go):

- create `pypirc` config file

```
[distutils]
index-servers =
  pypi
  pypitest

[pypi]
repository=https://pypi.python.org/pypi
username=your_username
password=your_password

[pypitest]
repository=https://testpypi.python.org/pypi
username=your_username
password=your_password
```

Then test the file on the `pypi` test server.

```
# test registration
python setup.py register -r pypitest

# test upload
python setup.py sdist upload -r pypitest
```

Finally submit

```
# for registering the package
python setup.py register -r pypi
```

```
# for actually uploading it
python setup.py sdist upload -r pypi
python setup.py bdist_wheel upload -r pypi
```

---

[Parallelizing Word2Vec in Multi-Core and Many-Core Architectures](https://arxiv.org/abs/1611.06172)

---

[The XSS Game by Google](https://xss-game.appspot.com/)

---

Reg Braithwaite on Optimism

[Optimism Talk](https://github.com/raganwald/presentations/blob/master/optimism.md)

[Optimism Essay](https://github.com/raganwald-deprecated/homoiconic/blob/master/2009-05-01/optimism.md#readme)

---

On your future and lottery/self improvement

[Winning the Lottery](https://www.youtube.com/watch?v=l_F9jxsfGCw&feature=youtu.be)

[smbc](http://www.smbc-comics.com/?id=2722)

---

[How to Write a 20 Page Research Paper in Under a Day](http://www.nerdparadise.com/litcomp/silly/20pagepaper/)

---

[DIY Computer Science](http://bradfieldcs.com/diy/)

---

[Stop saying learning to code is easy](http://www.hanselman.com/blog/StopSayingLearningToCodeIsEasy.aspx)

---

[Katherine Kirk - Keynote : Why Team Happiness can be the Worst Thing to Aim For](https://vimeo.com/143894732)

---

[War on Stupid People](http://www.theatlantic.com/magazine/archive/2016/07/the-war-on-stupid-people/485618/?single_page=true)

---

[We only hire the trendiest](http://danluu.com/programmer-moneyball/)

---

[On asking job candidates to code](http://philcalcado.com/2016/03/15/on_asking_job_candidates_to_code.html)

---

[A thing worth doing](https://www.chesterton.org/a-thing-worth-doing/)

---

[SBT without tests](http://stackoverflow.com/questions/26499444/how-run-sbt-assembly-command-without-tests-from-command-line)

Example for Windows:

    sbt "set test in assembly := {}" clean assembly

Example for Mac:

    sbt 'set test in assembly := {}' clean assembly

---

[Submitting Applications for Spark](http://spark.apache.org/docs/latest/submitting-applications.html)

---

[SpeechRecognition](https://pypi.python.org/pypi/SpeechRecognition/)

---

[How to cultivate the art of Serendipity](http://www.nytimes.com/2016/01/03/opinion/how-to-cultivate-the-art-of-serendipity.html?smid=fb-nytopinion&smtyp=cur&_r=1)

---

[Odds Are, It's Wrong](https://www.sciencenews.org/article/odds-are-its-wrong)

---

Taking Agile too extreme results in this:

<https://www.youtube.com/watch?v=a-BOSpxYJ9M>

<http://pragdave.me/blog/2014/03/04/time-to-kill-agile/>

---

[Perkson's Paradox](http://mathemathinking.blogspot.com.au/2014/10/berksons-paradox.html)

---

[Predictive Learning Via Rule Ensembles](http://statweb.stanford.edu/~jhf/ftp/RuleFit.pdf)

---

Notes with V:

**Whats the Difference between band 5 and 4?**

Both...

- drive best practise and continually improve

Band 5...

- is expected to upskill others
- contribute to solution (expection)

Band 4...

- expected to be technically strong and "do what you do well"

**What is a data science**

A combination of

1.  Communication and interpersonal skills
2.  Strong knowledge of ML and Statistics
3.  Consultancy skills

_Other Notes_

- Never ever stop learning and being curious with your technical skills.
- Focus on the business benefit; the monetary amoutn and impact.
- Remember your communication skills and the "D" space
- Articulate better how you interact with the stackholders (think about this before your interview)
- When you explain complex subjects always remember to make it simplier
- Keep future options open; both as a specialist and a manager
- Talk more about the strategies and goals in the business; be more proactive about finding what are the goals and painpoints - what exactly are we trying to solve?

---

PCA on "big data":

1.  Is it wide? Try randomized SVD
2.  Is it long? Try online methods for PCA

---

[What Can You Put in a Refrigerator?](http://prog21.dadgum.com/212.html)

---

Deep Learning with [Tensor Flow](http://tensorflow.org/) from Google.

---

[Living from one bag](http://zenhabits.net/lightly/)

---

[Markdeep](http://casual-effects.com/markdeep/)

---

[Alexander Coward](http://alumni.berkeley.edu/california-magazine/just-in/2014-09-02/cal-lecturers-email-students-goes-viral-why-i-am-not)

> Whatever you decide to do with your life, it???s going to be really, really complicated.
>
> Science and technology is complicated. History and politics is complicated. People are complicated. Figuring out how to be happy, and do simple things like take care of our kids and maintain friendships and relationships, is complicated.
>
> In order for you to navigate the increasing complexity of the 21st century you need a world-class education, and thankfully you have an opportunity to get one. I don???t just mean the education you get in class, but I mean the education you get in everything you do, every book you read, every conversation you have, every thought you think.
>
> You need to optimize your life for learning.
>
> You need to live and breath your education.
>
> You need to be _obsessed_ with your education.
>
> Do not fall into the trap of thinking that because you are surrounded by so many dazzlingly smart fellow students that means you???re no good. Nothing could be further from the truth.
>
> And do not fall into the trap of thinking that you focusing on your education is a selfish thing. It???s not a selfish thing. It???s the most noble thing you could do.
>
> Society is investing in you so that you can help solve the many challenges we are going to face in the coming decades, from profound technological challenges to helping people with the age old search for human happiness and meaning.
>
> That is why I am not canceling class tomorrow. Your education is really really important, not just to you, but in a far broader and wider reaching way than I think any of you have yet to fully appreciate.

---

[How to do Philosophy](http://www.paulgraham.com/philosophy.html)

[John Stuart Mill](http://plato.stanford.edu/entries/mill/)

---

[John Carmack on the art and science of programming/software engineering](https://blogs.uw.edu/ajko/2012/08/22/john-carmack-discusses-the-art-and-science-of-software-engineering/)

> the the NASA style devel??op??ment process, they can deliver very very low bug rates, but it???s at a very very low pro??duc??tiv??ity rate. And one of the things that you wind up doing in so many cases is cost ben??e??fit analy??ses, where you have to say, well we could be per??fect, but then we???ll have the wrong prod??uct and it will be too late."
>
> I???ve got??ten very big on the sta??tic analy??sis, I would like to be able to enable even more restric??tive sub??sets of lan??guages and restrict pro??gram??mers even more because we make mis??takes con??stantly.

---

[Norris Numbers](http://www.teamten.com/lawrence/writings/norris-numbers.html)

---

[Blur detection](http://www.pyimagesearch.com/2015/09/07/blur-detection-with-opencv/)

---

# The Gini Index

While quantile plots and double lift charts are easy to understand and interpret, they are also subjective. The Gini index, though it is more complex, has the benefit of boiling lift down to a single number.

The Gini index, named for Corrado Gini, is commonly used in economics to quantify national income inequality. Here is a Gini index plot for the United States per the Social Security Administration website:
Percentage of Earnings
We first sort the population based on income, from lowest to highest. The x-axis is the cumulative percentage of people and the y-axis is the cumulative percentage of earnings. The locus of those points forms what is known as the Lorenz curve, which is the dotted line in the graph above. For example, point A shows us that the poorest 60% of Americans earn about 20% of the income. The 45-degree line is called the Line of Equality, so named because, if everyone earned the same exact income, then the Lorenz curve would be the Line of Equality. However, everyone doesn???t earn the same income, and the Gini index is calculated as twice the area between the Lorenz curve and the Line of Equality.

How is the Gini index used to quantify model lift? It is constructed as follows:

1.  Sort policyholders from best to worst, as predicted by Model A.
2.  The x-axis is the cumulative percentage of exposure (car-years, house-years, etc).
3.  The y-axis is the cumulative percentage of losses.
4.  Calculate the Gini index for Model A as twice the area between the Lorenz Curve and the Line of Equality.
5.  Do the same for Model B and compare the Gini indices produced by the two models.

If Model A produced the Gini plot above, it would tell us that Model A has identified 60% of risks that contribute only 20% of total losses. A Gini index does not quantify the profitability of a particular rating plan, but it does quantify the ability of the rating plan to differentiate between the best and worst risks.

[source](http://www.casact.org/newsletter/index.cfm?fa=viewart&id=6540)

---

[Tufte and R](http://motioninsocial.com/tufte/)

---

[khan-crypto](https://www.khanacademy.org/computing/computer-science/cryptography)
[khan-animation](https://www.khanacademy.org/partner-content/pixar)

---

[Crypto](https://crypto.stanford.edu/~dabo/cryptobook/draft_0_2.pdf)

---

[Privacy tools](https://www.privacytools.io/)

---

[Switching to Linux](http://www.zdnet.com/article/sick-of-windows-spying-on-you-go-linux/):

> First, installing Mint is simple. You download the latest version of the operating system. Then, you put its ISO image on a USB stick using Pendrive Linux Universal USB Installer. You can also use the USB stick to try Mint without installing it. Just boot up from the USB stick and you'll get a good, albeit slow, look at Mint.

---

By Anne Rice (who I haven't read any books by and on her Facebook page):

> Signing off with thanks to all who have participated in our discussions of fiction writing today. I want to leave you with this thought: I think we are facing a new era of censorship, in the name of political correctness. There are forces at work in the book world that want to control fiction writing in terms of who "has a right" to write about what. Some even advocate the out and out censorship of older works using words we now deem wholly unacceptable. Some are critical of novels involving rape. Some argue that white novelists have no right to write about people of color; and Christians should not write novels involving Jews or topics involving Jews. I think all this is dangerous. I think we have to stand up for the freedom of fiction writers to write what they want to write, no matter how offensive it might be to some one else. We must stand up for fiction as a place where transgressive behavior and ideas can be explored. We must stand up for freedom in the arts. I think we have to be willing to stand up for the despised. It is always a matter of personal choice whether one buys or reads a book. No one can make you do it. But internet campaigns to destroy authors accused of inappropriate subject matter or attitudes are dangerous to us all. That's my take on it. Ignore what you find offensive. Or talk about it in a substantive way. But don't set out to censor it, or destroy the career of the offending author. Comments welcome. I will see you tomorrow.

?---

[CS for all](http://www.cs.hmc.edu/csforall/)

remember to check out their explanation for the halting problem.

---

[Art of simple living](http://www.redesignmyexistence.com/minimalist-living-tips-to-organize-your-life)
[Art of simple living - part 2](http://www.redesignmyexistence.com/minimalist-living-tips-to-organize-your-life-pt2)

---

**Fitness 2019**

Simply put, my fitness goal is to:

- Run more
- Lift more
- Address weaknesses

Currently, the most obvious weakness in my lifts are the bench press - hitting 1RPM of 100kg, should be the goal for 2019.

As for running goals, hitting sub 5 min per km for 6km should not only be doable, but possible in the near-ish future.

Other weakness in general fitness probably come in many forms such as:

- Handstands
- Mobility in ankles
