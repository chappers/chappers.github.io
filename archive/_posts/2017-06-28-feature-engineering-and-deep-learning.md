---
layout: post
category: code dump
tags: [python]
tagline:
---

In this post I will have a look at the first two projects within Udacity's self-driving car project. More specifically I will share some of my thoughts in a "meta-learning" sense; how do things that I know and current do relate to machine perception/deep learning problems.

1.  [Finding Lane Lines on the Road](https://github.com/udacity/CarND-LaneLines-P1)
2.  [Traffic Sign Recognition](https://github.com/udacity/CarND-Traffic-Sign-Classifier-Project)

The aspect of these projects which interest me much isn't so much the "deep learning" portion, but rather tackling problems which I will describe as _perception_ problems. In recent times deep learning has undoubtedly done well in perception problems; ones which human beings can not easily ascribe a rule to. Examples of these include:

- Playing games in the Deep Q Networks settings (e.g. AlphaGo)
- Image recognition; it isn't easy to describe why one picture is male or female in a rule-base framework
- All things speech; our language is complex without "nice" formal grammars in the same way that programming language parsers function

---

No matter what algorithm you use, hyper-parameter optimization will only do so much. Feature engineering and exploiting any knowledge that you might have is really important.

**Lane Lines**

The first challenge is finding lane lines. In this setting, if all we knew how to do was use edge detection algorithm, could we easily tackle this problem?

The types of knowledge we would impose could be:

- Expectation of where the lane lines will be. We would probably expect lane lines to be on the bottom half of the image
- Colours of the lane lines. Lane lines can probably be yellow or white - perhaps we can simply mask all other colours!

If we had the base image:

![png](/img/self-driving-cars-p1p2/p1-base.png)

We can apply some kind of trapezoidal shape which will be where we expect our lane lines to be:

![png](/img/self-driving-cars-p1p2/p1-lane-masking.png)

Using this image, we could try to change the yellow to white.

![png](/img/self-driving-cars-p1p2/p1-lane-white.png)

Finally we could use a magical line detection algorithm (generally hough transform) to complete this project.

![png](/img/self-driving-cars-p1p2/p1-lane-hough.png)

And overlay it on the original image!

![png](/img/self-driving-cars-p1p2/p1-lane-final.png)

Of course you can use some kind of linear interpolation to extend the line in an appropriate way.

**Traffic Sign Classification**

Even within the project description it suggested using LeNet will already yield a 89% accuracy score on the dataset.

It is important to realise that LeNet requires grayscale images. But is that sensible for traffic signs?

- Colour of the traffic sign is important for classification! (Use a Neural Net that takes in 3 colours)
- Traffic signs generally are high contrast so that it is easier to be viewed

Traffic signs can even be thought of some kind of hierarchical classification problems:

- If the sign has blue it will probably inform you of a direction
- If the sign is red is it a "no entry" sign
- If the sign is red and white it is a warning sign

Based on this the first thing we would do is to change LeNet to take in colour images. Luckily in TensorFlow (or Keras) this is as simple as changing what the input to accept an image with 3 channels.

Increasing contrast is also simple using openCV. Using some code which can be easily found online, one approach is to make use of the colour histograms to "spread" them out better using [contrast limited adaptive histogram equalization (CLAHE)](https://en.wikipedia.org/wiki/Adaptive_histogram_equalization#Contrast_Limited_AHE)

```python
def col_equalise(img):
    lab= cv2.cvtColor(img, cv2.COLOR_BGR2LAB)
    # split by channel
    l, a, b = cv2.split(lab)
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8,8))
    cl = clahe.apply(l)
    limg = cv2.merge((cl,a,b))
    final = cv2.cvtColor(limg, cv2.COLOR_LAB2BGR)
    return final
```

Before

![png](/img/self-driving-cars-p1p2/p2-base.png)

After

![png](/img/self-driving-cars-p1p2/p2-colcorrect.png)

This should immediately raise your validation accuracy to 91%.

Beyond that, regularization-like techniques could be used to further prevent overfitting in your training set (drop out parameter).

## On using TensorFlow GPU vs CPU

Based on a sample size of 1, I have found GPU performance to be at least 10x quicker than CPU.

For example, in this problem, the timing difference was around 40 secs for GPU and 9 minutes for CPU.
