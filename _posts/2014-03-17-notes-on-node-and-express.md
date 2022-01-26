---
layout: post
category: web micro log
tags: [node, express 3, express]
---

Express gets updated relatively frequently, so unfortunately tutorials go out of date
(and due to the nature of the internet, are updated infrequently). Inevitably, this
tutorial will also go out of date. Nevertheless, here are just some
notes from following online tutorials on building web applications using node and express.

# Getting Started

Assuming node is installed, you can go ahead and install all the necessary modules that
would be used for making a webapp. In this example the command would be:

    npm install -g express mongoose jade less expresso

Once that is done, you can then create a skeleton web app.

    express helloapp

Then you can run the app as usual using

    node app.js

## Changing things

You can change things on the fly without restarting the server. So just leave the server
running and see if we can change the default webpage.

### Routes

Looking at the file structure, we can probably "guess" that to change the index page, you
would alter `index.jade`. But what is `jade`? How does node know to locate `views/index.jade`?

Lets firstly look at the `app.js`.

```javascript
var express = require("express");
var routes = require("./routes");
var user = require("./routes/user");
```

What do these do?

Using what I know from the Python universe, the code above would be similar to

```javascript
import express
import routes
import routers.user as user
```

Where `routes` is located in the current working directory and express is the actual
package we installed earlier.

So where does the magic happen to generate the front page?

```javascript
app.get("/", routes.index);
```

What this "means" is in the base URL we will run the function in `routes.index`

Now switching to `./routes.index.js` (where it is directed), we can see that it goes to
`res.render('index', { title: 'Express' });`. We could guess this means _render_ the _response_ using `'index'` with some parameters.

Finally, looking at the `views folder` we can see `index.jade` which is what is
finally rendered. Change `index.jade` to whatever you want and refresh the page to get
it to show up.
