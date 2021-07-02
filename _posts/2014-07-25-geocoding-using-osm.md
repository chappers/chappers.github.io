---
layout: post
category : web micro log
tags : [r]
---

Using API calls is probably the easiest way to geocode address. Here is some sample R code which demonstrates how this can be done using two different sites:

```r
library(rjson)

address = "321 kent st, sydney,nsw 2000, australia"
format  = "json"
addressdetails = 0 


# nominatim API has a "fair usage" policy but is more or less unlimited
# you could in theory just install your own stack, and query that...
url = paste0("http://nominatim.openstreetmap.org/search?q=",
             gsub("[ ]+", "+", address),
             "&format=", format,
             "&addressdetails", addressdetails)

kentst <- fromJSON(file=url)
kentst[[1]]$lon
kentst[[1]]$lat

'
> kentst[[1]]$lon
[1] "151.2044631"
> kentst[[1]]$lat
[1] "-33.8677586"
'

# mapquest is actually unlimited, but you have to signup 
# for a free account, it too has a "fair usage" policy
# which means they might degrade the speed of your API calls
# it is under a community license (uses open data), though
# it is probably worth a further read. see terms of use.

API_KEY = "GO TO MAPQUEST TO GET YOUR API_KEY"

url1 <- paste0("http://open.mapquestapi.com/geocoding/v1/address?key=",API_KEY,
               "&inFormat=",format,
               "&location=",address)

kentst1 <- fromJSON(file=url1)

kentst1$results[[1]]$locations[[1]]$latLng

"
$lng
[1] 151.2045

$lat
[1] -33.86776
"
```

It is important to realse the constraints of this method, as it needs to lookup
every individual item. You will also have to be careful of the various use policies
for each method. 

Pay solutions are obviously better and more robust, but hopefully this provides a glimpse into what can be done cheaply if you don't have too much data. 
