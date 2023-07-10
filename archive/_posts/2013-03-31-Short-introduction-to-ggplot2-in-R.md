---
layout: post
category: code dump
tags: [R]
tagline:
---

`ggplot2` ([grammar of graphics](http://books.google.com/books?id=_kRX4LoFfGQC&lpg=PP1&dq=grammar%20of%20graphics&pg=PP1#v=onepage&q&f=false) plot) is a twist on the traditional way of displaying graphics, and will differ slighly compared with the `plot` functions in R.

## Layers

The general idea of GOG (grammar of graphics) is that graphics can be seperated into layers.

From Wickham's [paper](vita.had.co.nz/papers/layered-grammar.pdf), the layers can be summarised as follows:

- **data** or **aes**thetic mappings, which describe the variables (e.g. `x ~ y` plot)
- **geo**metric objects, which describe what you see, (e.g. line plot, boxplot, scatterplot)
- **scales**s which map values to things like colour, size, or shape.
- **facet** breaks data into subsets to display as small multiples

## Basic Plotting

The most basic way to use `ggplot2` is using `qplot` this is extremely similar to your normal `plot` function in R.

    with(mtcars, plot(cyl, mpg, pch = carb, col = gear))

![init-repo](/img/short-intro-ggplot2/base_plot.png)

    qplot(cyl, mpg, data=mtcars, shape=as.factor(carb), col=gear)

![init-repo](/img/short-intro-ggplot2/ggplot_basic.png)
Though this can be easily broken down into several layers.

    ggplot(mtcars, aes(cyl, mpg))+aes(shape=as.factor(carb), color=gear)+geom_point()

![init-repo](/img/short-intro-ggplot2/ggplot_basic.png)
Breaking things into layers is useful, because you can save the layers and generate different plots quite easily, if needed.

    p <- ggplot(mtcars, aes(cyl, mpg))
    p + geom_point()+facet_grid(carb~.)+aes(color=gear)

![init-repo](/img/short-intro-ggplot2/facet_gear.png)
p + geom_point()+facet_grid(.~gear)+aes(shape=as.factor(carb))

![init-repo](/img/short-intro-ggplot2/facet_carb.png)

## Adding Lines to Scatter Plots

Adding additional geometric objects is the same as adding an extra layer.

In base `R`

    with(mtcars, plot(cyl,mpg))
    with(mtcars, lines(lowess(cyl,mpg),col='red'))

![init-repo](/img/short-intro-ggplot2/base_line.png)

Under `ggplot2`

    p <- ggplot(mtcars, aes(cyl,mpg)) + geom_point()
    p + geom_smooth(method='loess')

![init-repo](/img/short-intro-ggplot2/ggplot_loess.png)

You can see we merely add an additional geometric layer which has our smoother.

We can even define our own smoothers if we wish:

    library(splines)
    p + geom_smooth(method='lm', formula=y~ns(x,2))

![init-repo](/img/short-intro-ggplot2/ggplot_spline.png)

That should be more than enough to get started!
