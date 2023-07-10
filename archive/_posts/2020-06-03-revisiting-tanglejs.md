---
layout: post
category:
tags:
tagline:
title: Revisiting TangleJS
---

Sometimes you want an interactive document; whereby you move a slider and values on the page changes. A good example of this is on [distill.pub](https://distill.pub/2017/momentum/) and of course [Tangle](http://worrydream.com/Tangle/).

<script src="https://unpkg.com/mithril@2.0.4/mithril.min.js"></script>

Let's look at some of these components and see what we can do with it in a more "modern" framework, seeing that the [library itself hasn't been updated in years!](https://github.com/worrydream/Tangle) At its heart, Tangle is essentially a fancy "range" input:

<input type="range" min="1" max="100" value="50">

And you want some javascript to do some computation off the values in the slider! The question then is: "How exactly do we do this?"

Let's take a look at [`mithril.js`](https://mithril.js.org):

<div id="m-slider1"></div>

<script>
var root = document.getElementById("m-slider1")
var val = 20 // setting val

var slider = {
  view: function (scope) {
    return m("main", [
        m("p", "range input: " + val),
        m("input[type=range]", {
            min:0,
            max:100,
            step:2,
            value: val,
            oninput: function(){val = this.value;}
        })
    ])
  }
}

m.mount(root, slider)
</script>

```html
<div id="m-slider1"></div>

<script>
  var root = document.getElementById("m-slider1");
  var val = 20; // setting val

  var slider = {
    view: function (scope) {
      return m("main", [
        m("p", "range input: " + val),
        m("input[type=range]", {
          min: 0,
          max: 100,
          step: 2,
          value: val,
          oninput: function () {
            val = this.value;
          },
        }),
      ]);
    },
  };

  m.mount(root, slider);
</script>
```

If we construct the Tangle document example:

<div id="m-slider2"></div>

<script>
var mslider2 = document.getElementById("m-slider2")
var cookies = 2 // setting val

var cookieCalculator = function(cookies) {return cookies*50}

var cookieSlider = {
  view: function (scope) {
    return m("main", [
        m("label", {
            for:"slider2"
        }, "Cookies: "+cookies),
        m("input[type=range]", {
            min:0,
            max:10,
            step:1,
            value: cookies,
            id: "slider2", 
            oninput: function(){cookies = this.value;}
        }),
        m("p", "When you eat "+ cookies + " cookies, you consume " + cookieCalculator(cookies) + " calories.")
    ])
  }
}

m.mount(mslider2, cookieSlider)
</script>

```html
<div id="m-slider2"></div>

<script>
  var mslider2 = document.getElementById("m-slider2");
  var cookies = 2; // setting val

  var cookieCalculator = function (cookies) {
    return cookies * 50;
  };

  var cookieSlider = {
    view: function (scope) {
      return m("main", [
        m(
          "label",
          {
            for: "slider2",
          },
          "Cookies: " + cookies
        ),
        m("input[type=range]", {
          min: 0,
          max: 10,
          step: 1,
          value: cookies,
          id: "slider2",
          oninput: function () {
            cookies = this.value;
          },
        }),
        m(
          "p",
          "When you eat " +
            cookies +
            " cookies, you consume " +
            cookieCalculator(cookies) +
            " calories."
        ),
      ]);
    },
  };

  m.mount(mslider2, cookieSlider);
</script>
```

Unfortunately it doesn't look as pretty as we would like - more on that next time!
