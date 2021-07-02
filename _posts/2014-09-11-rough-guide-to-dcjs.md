---
layout: post
category : web micro log
tags : [js]
---

[dc.js](http://dc-js.github.io/dc.js/) is quite a nice library which allows you to filter of multiple graphs intuitively.

Surprisingly it is not too difficult to develop on. If you have an understanding of `.dimensions` and `.group()`, you're basically ready to go and start creating beautiful interactive dashboards. Below I have a simple example which I will quickly describe, particularly the parts of the library which I really like.

## Exploring the Code

`.dimension` allows you to easily define how you would like the "cut" the data. For the bar graph for example, I decided to cut it by age, split into groups of 5 years. This was easily achieved using: `var age_dim = ndx.dimension(function(d) { return Math.round(d.age/5)*5 })`

`.group` allows you to specify how the groupings might be done. You may sum up the values or simply count the observations. Here we have just counted the observations by using `age_dim.group()`.

Plotting both age and gender graphs simply meant specifying the type of graph (`barChart` or `pieChart`) and adding the properties. This was all then rendered using `dc.renderAll()`. Although not shown here you can attach events allowing dynamic tables with the data to be displayed (or any other useful information) which can be helpful for people seeking to have a visual way to filter their data.

</br>
</br>

<div id="chart-bar" ></div>
<div id="chart-pie" ></div>

</br>
</br>

</br>
</br>

</br>
</br>

</br>
</br>

</br>
</br>

</br>
</br>


<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
<script src="http://d3js.org/d3.v3.min.js" charset="utf-8"></script>
<script type='text/javascript' src="http://tvinci.github.io/webs/js/crossfilter.js"></script>
<script type='text/javascript' src="http://cdnjs.cloudflare.com/ajax/libs/dc/1.7.0/dc.js"></script>

<script>
$('head').append('<link rel="stylesheet" href="http://cdnjs.cloudflare.com/ajax/libs/dc/1.7.0/dc.css" type="text/css" />');

var data = []

for (var i=0;i<250;i++) {
  var d = {}
  d.sex = Math.random() > 0.5 ? "Male" : "Female";
  d.age = ((((Math.random() + Math.random() + Math.random() + Math.random() + Math.random() + Math.random()) - 3) / 3)*40) +45;
  data[i] = d
}

/*
var data = [{'sex':'Male', 'age': 10},
{'sex':'Female', 'age': 25},
{'sex':'Female', 'age': 32},
{'sex':'Male', 'age': 28},
]
*/
var ndx = crossfilter(data); 

// age plot
var age_dim = ndx.dimension(function(d) { return Math.round(d.age/5)*5 }) 
var age_grp = age_dim.group();
var agechart = dc.barChart("#chart-bar");

agechart
.width(500)
.height(300)
.dimension(age_dim)
.group(age_grp)
.round(Math.floor)
.centerBar(true)
.gap(1)
.x(d3.scale.linear().domain([15,80]))
.elasticY(true)
.filter([25,35])
.xAxisLabel("Age")
.xAxis();

agechart.xUnits(function(){return 14;});

// gender graph
var sex_dim = ndx.dimension(function(d) {return d.sex});
var sex_count = sex_dim.group();

var sexchart = dc.pieChart("#chart-pie");

sexchart
.width(150)
.height(150)
.dimension(sex_dim)
.group(sex_count)
.innerRadius(30);

dc.renderAll()
</script>
