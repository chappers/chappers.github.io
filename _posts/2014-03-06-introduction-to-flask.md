---
layout: post
category : web micro log
tags : [python]
---

Flask is not something I haven't heard of before, but I thought I'll have a look at this microframework
just to see whats different, and perhaps learn a different way to build web applications.

# Getting Started

Lets begin with the typical "Hello World" example.

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def get_tasks():
    return "Hello World"
    
if __name__ == '__main__':
    app.run(debug=True, port=8080)
```

This isn't anything overly complicated or magical. Though when compared with some other frameworks this is beautifully simplistic.

## Trial and Error

Maybe now with just the above you can try a few new things. How about, creating a new page by changing the route?

```python
from flask import Flask

app = Flask(__name__)

some_string = "A string"

@app.route('/string')
def get_tasks():
    return some_string

if __name__ == '__main__':
    app.run(debug = True, port=8080)
```

What happens when you try to navigate to `127.0.0.1:8080`? What about `127.0.0.1:8080/string`?

# Variable Rules

```python
from flask import Flask

app = Flask(__name__)

@app.route('/<capitalise>')
def get_tasks(capitalise):
    return capitalise.upper()

if __name__ == '__main__':
    app.run(debug = True, port=8080)
```

You can make use of variable rules to apply things...especially with converters.
Read more on the main site (or the [quickstart](http://flask.pocoo.org/docs/quickstart/) guide)

# HTTP Methods

Before we explore HTTP methods and forms lets look at templates. Templates in flask is quite simple, just create
a new folder called "templates" and throw your html templates there.

In order to demonstrate http methods, create a form to actually enter your information. 

```html
<html>
<head>
<title>To Do List</title>
</head>
<body>
<form action="/todo" method="post">
<input type="text" name="new_item"></input>
<input type="submit" value="Add"></input>
</form>
</body>
</html>
```

Then, it is rather easy to extract this information.

```python
from flask import Flask, render_template, request, redirect
app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/todo', methods = ['POST'])
def todo():
    item = request.form['new_item']
    print(item)
    return redirect('/')
   
if __name__ == '__main__':
    app.run(debug=True, port=8080)
```

Of course the next step is how would you get that information? How can you use it?

Create a new template to print out all the printed out everything in the to do list.

```html
<html>
<head>
<title>All Item</title>
</head>
<body>
<ul>
\{% for el in todo %}
<li>{{ el }}</li>
\{% endfor %}
</ul>
</body>
</html>
```

Now changing the python file:

```python
#!flask/bin/python
from flask import Flask, render_template, request, redirect
app = Flask(__name__)

todo_list = []

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/todo', methods = ['POST'])
def todo():
    item = request.form['new_item']
    todo_list.append(item)
    print(item)
    return redirect('/')

@app.route('/result')
def result():
    return render_template('entered_info.html', todo=todo_list)
    
if __name__ == '__main__':
    app.run(debug=True, port=8080)
```
    
Now if you run the server, you will see all the items that you pass through the form.




