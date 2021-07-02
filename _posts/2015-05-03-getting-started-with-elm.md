---
layout: post
category : web micro log
tags : 
---


[elm](http://elm-lang.org/) is an interesting language which compiles to javascript. It has many advance features (which I am definitely not across). I was trying to integrate elm into Cordova, so here are some of my notes on how to pull together an example webpage.

# Running an example

Consider the simple markdown [example](http://elm-lang.org/edit/examples/Elements/Markdown.elm):

```py

import Markdown


main = Markdown.toElement """

# Welcome to Elm

This [markdown](http://daringfireball.net/projects/markdown/)
parser lives in the [elm-markdown][] package, allowing you to
generate `Element` and `Html` blocks.

[elm-markdown]: http://package.elm-lang.org/packages/evancz/elm-markdown/latest

"""

```

If you simply save a file called "markdown.elm", then you're (almost) done. A few things which we have to consider is what happens when you try to make an html file.

```bat
elm-make markdown.elm
```

It will complain that some of the dependencies are missing. After some search you'll realise that in order to add the dependencies we will have to add it to `elm-package.json` file which will be created. 

```
"evancz/elm-markdown": "1.0.0 <= v < 4.0.0"
```

If we try to run `elm-make` again it will tell you to install using `elm-package install`. Doing this will install the require package. 

Finally if we run `elm-make` we will produce a single file `elm.js`.

We can create a simple html document which uses this file in the following manner:

```html

<body>
<script src="elm.js"></script>
<script type="text/javascript">Elm.fullscreen(Elm.Main)</script>
</body>

```

And then you're done! You have a working elm project. 

Alternatively you could directly create html output by using 

```bat
elm-make markdown.elm --output markdown.html
```

Enjoy playing around with elm!

<iframe width="640" height="360" src="https://www.youtube.com/embed/39Fk513CWOU" frameborder="0" allowfullscreen></iframe>

