---
layout: post
category : code dump
tags : [python, deck.js]
tagline: 
---

_update: check out [Puma.js](http://chappers.github.io/puma/) which uses the same ideas but with no server-side compilation needed._

I've been interested in various forms of markdown for a while (after all jekyll uses markdown heavily). Thats when I've discovered [deck.js](http://imakewebthings.com/deck.js/). 

A very short python script will allow you to generate presentations without having to worry about layout. The structure of the presentations is using markdown as normal and seperating each slide by `---`. For example, the markdown presentation to generate this [presentation](https://googledrive.com/host/0ByHWFFfBDxCFZ1ctMk9MRjBXc0U/deckjs/presentation.html) is as below:

	#Chapman Siu
	---
	Chapman's personal:  

	*	projects, essays and code dump
	*	web micro log
	*	code repository
	---
	#What's Here?  

	I have my solutions for selected [dailyprogrammer](http://www.reddit.com/r/dailyprogrammer) [challenges](code.html).  
	These are written in a variety of languages including `C++`, `python`, and `ruby`.

	Some `SAS` code is provided which is provided on uncopyright basis. They are a combination of experiments which may be beneficial for some other `SAS` users out there who use `SAS` for text mining. However I would highly recommend a scripting language such as `ruby` or `python` to fulfill your needs.  

	I have found `regex` to be invaluable, with [Rubular](http://http://rubular.com/) being my choice of website for testing `regex` expressions.
	---
	#Mathematics

	$$f(0) = 0$$
	$$f(1) = 1$$
	$$f(0.2) = 0.8$$

	<!-- mathjax --> 
	<script type="text/javascript"
	src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
	</script>

The python code to generate this is a simple few line solution, since slides are basic html.

	import markdown
	import jinja2
	import re
	import os

	template_dir = os.path.join(os.path.dirname(__file__), 'templates')
	jinja_env = jinja2.Environment(loader=jinja2.FileSystemLoader(template_dir), autoescape=True)

	f = open(os.path.join(os.path.dirname(__file__), 'presentation.md')).read()
	m = [markdown.markdown(x.strip()) for x in re.split('---+',f)]

	p = jinja_env.get_template("base.html").render(slides = m)

	outputFile = open(os.path.join(os.path.dirname(__file__), 'presentation.html'),'w').write(p)

Hopefully this will provide a nice outline to how to generate protypes for presentations quickly and effectively. I know I will certainly use `deck.js` in the future for rapid prototyping. 

Would I use it as a final product? I'm not too sure. The reason would be more related to industry expectations and whether other people can open this file or edit it. 

Other frameworks that I will look into would be [`impress.js`](http://bartaz.github.com/impress.js/), which certainly is impressive.
