---
layout: post
category : web micro log
tags : [js]
---

Recently I have been looking at a visualisation grammar called [Vega](http://trifacta.github.io/vega/). The main reason is the simplicity with generating graphs for the web when you come from other non-web areas such as Python.

But why Vega? The visualisation grammar is something which I believe is easily accessible for people who don't have knowledge in HTML or JavaScript. [D3](http://d3.js.org) although a wonder and powerful library, is way too difficult to even do the simplest of actions, whilst many of the other libraries seem to connect to D3, removing parts of its abstractions. Though they are useful, I truly believe that an approach like Vega is the future. (I am also highly influenced by [R's ggvis](http://ggvis.rstudio.com/) project which uses Vega as well).

As of writing there are no tooltips on the graphs. Though the authors have noted that it is slated for a future release when they flesh out the grammar for developing tooltips. Nevertheless I thought about the easiest and painless way (though not necessarily the best!) is to add my own tooltips, and concluded that just using jQuery with CSS is the easiest way to add tooltips. 

## The Tooltip

To create the tooltip, all we need is some text which follows our mouse when it hovers over the suitable part of the graph. 

1. Build a box which follows our mouse.

To do this we will use jQuery to continually update the position of the mouse and place the tooltip relative to the mouse. 

```html
  <head>
    <style>
/* this is for tooltips in the graph*/
#tooltip {
    position: absolute;
    float: left;
    background-color: #fff;
    border: 2px solid #aaa;
    padding: 2px;
    border-radius: 5px;
}
    </style>
    
  </head>
  <body>
    <div id="tooltip">This is a tooltip</div>
    <script src="http://code.jquery.com/jquery-1.10.2.js"></script>
    <script>
    $(document).bind('mousemove', function(e){
        $('#tooltip').css({
           "left":  e.pageX + 20,
           "top":   e.pageY
        });
    });
    </script>
  </body>   
```

Expanding on this idea, we can easily add it to our graph which is in Vega. The trick is that in the "mouseover" even we will set all the styling and the words in the toolbox, whilst on "mouseout" we will remove all the styling and text in the toolbox thus "hiding" the toolbox.

```js
vg.parse.spec(spec, function(chart) {
  var view = chart({el:"#view"})
    .on("mouseover", function(event, item) {   
      // set the css for the tooltip
      $('#tooltip').text("Value is : "+item.datum.data.y);
      $('#tooltip').css("border", "2px solid #aaa");
      $('#tooltip').css("padding", "2px");
      $('#tooltip').css("border-radius", "5px");
    })
    .on("mouseout", function(event, item) {
      // remove all styling from the tooltip
      $('#tooltip').text("");
      $('#tooltip').css("border", "0px");
      $('#tooltip').css("padding", "0px");
      $('#tooltip').css("border-radius", "0px");
    })
    .update();
  });
```

The result can be seen in the graph below:

<div id="view" class="view"></div>
<div id="tooltip"></div>

<script src="http://trifacta.github.io/vega/lib/d3.v3.min.js"></script>
<script src="http://trifacta.github.io/vega/vega.js"></script>
<script src="http://code.jquery.com/jquery-1.10.2.js"></script>
<script type="text/javascript">
var spec =  
{
"width": 400,
"height": 200,
"padding": {"top": 10, "left": 30, "bottom": 30, "right": 10},
"data": [
{
"name": "table",
"values": [
{"x": 1,  "y": 28}, {"x": 2,  "y": 55},
{"x": 3,  "y": 43}, {"x": 4,  "y": 91},
{"x": 5,  "y": 81}, {"x": 6,  "y": 53},
{"x": 7,  "y": 19}, {"x": 8,  "y": 87},
{"x": 9,  "y": 52}, {"x": 10, "y": 48},
{"x": 11, "y": 24}, {"x": 12, "y": 49},
{"x": 13, "y": 87}, {"x": 14, "y": 66},
{"x": 15, "y": 17}, {"x": 16, "y": 27},
{"x": 17, "y": 68}, {"x": 18, "y": 16},
{"x": 19, "y": 49}, {"x": 20, "y": 15}
],
"transform": [{"type":"filter", "test":"d.data.x>=0"}]
}

],
"scales": [
{
"name": "x",
"type": "ordinal",
"range": "width",
"domain": {"data": "table", "field": "data.x"}
},
{
"name": "y",
"range": "height",
"nice": true,
"domain": {"data": "table", "field": "data.y"}
}
],
"axes": [
{"type": "x", "scale": "x"},
{"type": "y", "scale": "y"}
],
"marks": [
{
"type": "rect",
"from": {"data": "table"},
"properties": {
"enter": {
"x": {"scale": "x", "field": "data.x"},
"width": {"scale": "x", "band": true, "offset": -1},
"y": {"scale": "y", "field": "data.y"},
"y2": {"scale": "y", "value": 0}
},
"update": {
"fill": {"value": "steelblue"}
},
"hover": {
"fill": {"value": "red"}
}
}
}
]
}

$('#tooltip').css({
"position": "absolute",
"float": "left",
"background-color": "#fff"
});

$(document).bind('mousemove', function(e){
$('#tooltip').css({
"left":  e.pageX + 20,
"top":   e.pageY
});
});

vg.parse.spec(spec, function(chart) {
var view = chart({el:"#view"})
.on("mouseover", function(event, item) {   
// set the css for the tooltip
$('#tooltip').text("Value is : "+item.datum.data.y);
$('#tooltip').css({
"border": "2px solid #aaa",
"padding": "2px",
"border-radius": "5px"})
})
.on("mouseout", function(event, item) {
// remove all styling from the tooltip
$('#tooltip').text("");
$('#tooltip').css({
"border": "0px",
"padding": "0px",
"border-radius": "0px"})
})
.update();
});
</script>




