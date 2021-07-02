---
layout: post
category : code dump
tags : [d3, jquery]
tagline: 
---

Here is an outline of how I managed to use SVG elements 'responsive'. I am be no means an expert by javascript, infact I would actually say I'm a [copy and paste programmer](http://www.codinghorror.com/blog/2009/04/a-modest-proposal-for-the-copy-and-paste-school-of-code-reuse.html) when it comes to javascript. 

My trail of though on how you could make SVG elements reponsive is through jQuery. More specifically through [`$(window).resize`](http://api.jquery.com/resize/).

After defining a function which will draw the SVG element based on the browser size, then `$(window).resize` function will redraw the svg appropriately. 

	$(window).resize(function() {
		draw();
	});

Is this the most optimal solution? Probably not. But it works! Until I expand my skillset this is how I would approach the problem.

A working example is located [here](https://googledrive.com/host/0ByHWFFfBDxCFZ1ctMk9MRjBXc0U/venn-diagram.html). This is based on an example provided by [Helen Lu from Stanford University](http://stanford.edu/~helenlu/cs448b/).

The next thing is to get it working with bootstrap.