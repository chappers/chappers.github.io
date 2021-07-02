---
layout: post
category : web micro log
tags : [python]
---

Announcing [`formdown`](http://charliec443.github.io/formdown/). This is a simple python module which assists with generating pesky forms without all the html tags. 

This can be combined with [datepicker](http://jqueryui.com/datepicker/).

Example:

```python
form = """name=____
date:datepicker=_____"""

print formdown.parse_form(form)
```

Output

```html
<p><label>Name</label><input type="text" name="name" id="name" /></p>
<p><label>Date</label><input type="text" name="date" id="date" class="datepicker"/></p>
<p><input type="submit" value="Submit"></p>
```

Note that since we have `class="datepicker"`, we will need to change the sample on the homepage:

```js
$(function() {
  $( ".datepicker" ).datepicker();
});
```

Hopefully this can be helpful!