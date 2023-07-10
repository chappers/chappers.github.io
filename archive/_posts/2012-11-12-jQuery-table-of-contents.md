---
layout: post
category: code dump
tags: [jquery, responsive web, skeleton]
tagline:
---

Based on Janko's [jQuery table of contents](http://www.jankoatwarpspeed.com/post/2009/08/20/Table-of-contents-using-jQuery.aspx).

<del>Repository: [github repository](https://github.com/chappers/ToC-using-jQuery), using [Skeleton](http://www.getskeleton.com/) as a basis for responsive web design.</del>

<del>[Click here](http://htmlpreview.github.com/?https://github.com/chappers/ToC-using-jQuery/blob/master/markdownText.html) to view it in action.</del>

_update:_ _I've moved and refactored my code [here](https://github.com/chappers/Contents) this post will be kept for legacy, though the code is more or less the same as before (23 February 2013)_

##Goals

Create something which was independent of css (in the sense that you have to manually add addition css code in the html file)
Create something which would fit with responsive web design
Although I can see many portions which can still be improved, this is my edited version of [Janko's jQuery ToC](http://www.jankoatwarpspeed.com/post/2009/08/20/Table-of-contents-using-jQuery.aspx).

    $(document).ready(function() {
    	$("h1, h2, h3").each(function(i, value) {
    		var current = $(this);
    		current.attr("id", "title" + i);
    		if (current.attr("tagName") == 'H1') {
    			$("#toc").append("<ul><li><a id='link" + i + "' href='#title" + i + "' title='" + current.attr("tagName") + "'>" + current.html() + "</a></li></ul>");
    		}
    		else if (current.attr("tagName") == 'H2') {
    			$("#toc").append("<ul><ul><li><a id='link" + i + "' href='#title" + i + "' title='" + current.attr("tagName") + "'>" + current.html() + "</a></li></ul></ul>");
    		}
    		else if (current.attr("tagName") == 'H3') {
    			$("#toc").append("<ul><ul><ul><li><a id='link" + i + "' href='#title" + i + "' title='" + current.attr("tagName") + "'>" + current.html() + "</a></li></ul></ul></ul>");
    		}
    	});
    });

Added static nav bar thanks to [stackoverflow](http://stackoverflow.com/questions/13190320/fixed-sidebar-with-skeleton-responsive-layout).
