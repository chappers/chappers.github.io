---
layout: post
category : web micro log
tags : [r, tableau]
---

<script type='text/javascript' src='http://public.tableausoftware.com/javascripts/api/viz_v1.js'></script><div class='tableauPlaceholder' style='width: 1004px; height: 869px;'><noscript><a href='#'><img alt='Dashboard 1 ' src='http:&#47;&#47;public.tableausoftware.com&#47;static&#47;images&#47;Re&#47;RecordedCrimeNSW1995-2009&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz' width='1004' height='869' style='display:none;'><param name='host_url' value='http%3A%2F%2Fpublic.tableausoftware.com%2F' /> <param name='site_root' value='' /><param name='name' value='RecordedCrimeNSW1995-2009&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='http:&#47;&#47;public.tableausoftware.com&#47;static&#47;images&#47;Re&#47;RecordedCrimeNSW1995-2009&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /></object></div><div style='width:1004px;height:22px;padding:0px 10px 0px 0px;color:black;font:normal 8pt verdana,helvetica,arial,sans-serif;'><div style='float:right; padding-right:8px;'><a href='http://www.tableausoftware.com/public/about-tableau-products?ref=http://public.tableausoftware.com/views/RecordedCrimeNSW1995-2009/Dashboard1' target='_blank'>Learn About Tableau</a></div></div>

I thought I would have a look at some open data and I came across the crime data set. The data used is:

*  [Recorded Crime Dataset (NSW)](http://data.nsw.gov.au/data/dataset/recorded-crime-dataset-nsw/resource/1046c49f-c896-4831-ab06-a26c8666f01f)  
*  [Postcode 2011 to Local Government Area 2011 ](http://www.abs.gov.au/AUSSTATS/abs@.nsf/DetailsPage/1270.0.55.006July%202011?OpenDocument)

# Using R

I thought this was a good opportunity to practise some R skills, though it could have easily been accomplished in Python as well. In order to read the data into R "nicely" I saved the relevant data into `csv` format. There wasn't really anything tricky, though I did discover this neat code in [stats.stackexchange](http://stats.stackexchange.com/a/7885)

```r
agg <- by(dx, dx$ID, FUN = function(x) x[1, ])
# Which returns a list that you can then convert into a data.frame thusly:
do.call(rbind, agg)
```

What this code does is take the first observation by a particular grouping (similar to `proc sort nodupkey` in `SAS`).

[Github Repo](https://github.com/charliec443/NSW-Crime-1995-2009)

# Using Tableau

I made a screencast on how I used Tableau. The video is cut it out rather abruptly and the sound quality isn't great, but I suppose you just have to made do with what you've got! Overall this project was far simpler than what I imagined and I'm reasonably pleased with the results. 

<iframe width="640" height="360" src="//www.youtube.com/embed/yFrE7lLPjz8" frameborder="0" allowfullscreen></iframe>

