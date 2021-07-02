---
layout: post
category : web micro log
tags : 
---

I'm currently looking at the Udacity course called [Data Visualization and D3.js](https://www.udacity.com/course/ud507). Working through Lesson 2 involves recreating some visualisations which were "badly" made on the internet. So here are some of the ones which I've come up with using [dimple.js](http://dimplejs.org).

Firstly lets consider the image showing the sales of various fast food restaurants:

![fastfood](/img/dimple-data/fastfood.jpg)

We can see that firstly the graphics although attempt to be to scale, it is very difficult to determine if the Taco Bell logo is indeed half the size of the Pizza Hut logo. Secondly the scale of the right is largely redundant, but either way this is not a strong graphic. Below is my attempt to recreate it using dimple.

<div id="fastfood"></div>

Next up we have MLS salaries. 

![mls](/img/dimple-data/mls.jpg)

Now I haven't got the same data, nor have I thoroughly cleansed it, however I believe grouping them in such a dense way is deterimental. Rather keeping it simple would be more more effective. If there was a need to see the individual's earnings a table below the graphic would be optimal. 

<div id="mls"></div>

The next few are from Fox, which all show an altered scale which makes no sense. They are shown below without comentary, since they all seem to follow the trend of breaking common sense.

![jobloss](/img/dimple-data/jobloss.jpg)

<div id="jobloss"></div>

![unemployment](/img/dimple-data/unemployment.jpg)

<div id="unemployment"></div>

![border](/img/dimple-data/border.jpg)

<div id="border"></div>

<script src="http://d3js.org/d3.v3.min.js"></script>
<script src="http://dimplejs.org/dist/dimple.v2.0.0.min.js"></script>

<script>
function drawFF(data) {
"use strict";
var margin = 75,
width = 575 - margin,
height = 400 - margin;

/*add the title to the plot.*/
d3.select("#fastfood")
.append("h2")
.text("Fast Food Sales");

var svg = d3.select("#fastfood")
.append("svg")
.attr("width", width + margin)
.attr("height", height + margin)
.append('g')
.attr('class','chart');
/*
Dimple.js Chart construction code
*/

var myChart = new dimple.chart(svg, data);
var x = myChart.addCategoryAxis("x", "restaurant");           
var y = myChart.addMeasureAxis("y", "sales");          


y.tickFormat = ',.f'; // add the decimals in the tooltip
myChart.addSeries(null, dimple.plot.bar);          
myChart.draw();

x.titleShape.text("Restaurant");
y.titleShape.text("Sales (in Billions)");
}

function drawMLS(data) {

/*
D3.js setup code
*/

"use strict";
var margin = 75,
width = 575 - margin,
height = 400 - margin;

/*add the title to the plot.*/
d3.select("#mls")
.append("h2")
.text("2014 MLS Player Salaries");

var svg = d3.select("#mls")
.append("svg")
.attr("width", width + margin)
.attr("height", height + margin)
.append('g')
.attr('class','chart');

/*
Dimple.js Chart construction code
*/
var myChart = new dimple.chart(svg, data);
var x = myChart.addCategoryAxis("x", "Club");           
var y = myChart.addMeasureAxis("y", "Base Salary");          

y.tickFormat = ',.f'; 

myChart.addSeries("Position", dimple.plot.bar);          
myChart.draw();

//x.titleShape.text("Restaurant");
//y.titleShape.text("Sales (in Billions)");

};

function drawJL(data) {

/*
D3.js setup code
*/

"use strict";
var margin = 75,
width = 575 - margin,
height = 400 - margin;

/*add the title to the plot.*/
d3.select("#jobloss")
.append("h2")
.text("Job Loss By Quarter (according to Fox News)");

var svg = d3.select("#jobloss")
.append("svg")
.attr("width", width + margin)
.attr("height", height + margin)
.append('g')
.attr('class','chart');

/*
Dimple.js Chart construction code
*/
var myChart = new dimple.chart(svg, data);
var x = myChart.addTimeAxis("x", "Year");           
var y = myChart.addMeasureAxis("y", "Job Loss");
x.tickFormat = "%Y";
y.tickFormat = ',.f'; 

myChart.addSeries(null, dimple.plot.line);
myChart.addSeries(null, dimple.plot.scatter);
myChart.draw();

//x.titleShape.text("Restaurant");
//y.titleShape.text("Sales (in Billions)");

};

function drawEM(data) {

/*
D3.js setup code
*/

"use strict";
var margin = 75,
width = 575 - margin,
height = 400 - margin;

/*add the title to the plot.*/
d3.select("#unemployment")
.append("h2")
.text("Job Loss By Quarter (according to Fox News)");

var svg = d3.select("#unemployment")
.append("svg")
.attr("width", width + margin)
.attr("height", height + margin)
.append('g')
.attr('class','chart');

/*
Dimple.js Chart construction code
*/
var myChart = new dimple.chart(svg, data);
var x = myChart.addTimeAxis("x", "Date");           
var y = myChart.addMeasureAxis("y", "Employment Rate");
x.tickFormat = "%d-%Y";
y.tickFormat = ',.f'; 

myChart.addSeries(null, dimple.plot.line);
myChart.addSeries(null, dimple.plot.scatter);
myChart.draw();

//x.titleShape.text("Restaurant");
//y.titleShape.text("Sales (in Billions)");

};

function drawB(data) {

/*
D3.js setup code
*/

"use strict";
var margin = 75,
width = 575 - margin,
height = 400 - margin;

/*add the title to the plot.*/
d3.select("#border")
.append("h2")
.text("Southwest Border Apprehensions");

var svg = d3.select("#border")
.append("svg")
.attr("width", width + margin)
.attr("height", height + margin)
.append('g')
.attr('class','chart');

/*
Dimple.js Chart construction code
*/
var myChart = new dimple.chart(svg, data);
var x = myChart.addCategoryAxis("x", "Year");           
var y = myChart.addMeasureAxis("y", "Border Apprehensions");
y.tickFormat = ',.f'; 

myChart.addSeries(null, dimple.plot.bar);
myChart.draw();

};

</script>

<script>
d3.tsv("/img/dimple-data/fastfood.tsv", drawFF);
d3.tsv("/img/dimple-data/mls_salaries_2014.tsv", drawMLS);
d3.tsv("/img/dimple-data/jobloss.tsv", drawJL);
d3.tsv("/img/dimple-data/employment.tsv", drawEM);
d3.tsv("/img/dimple-data/border.tsv", drawB);
</script>
