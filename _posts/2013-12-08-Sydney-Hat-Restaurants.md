---
layout: post
category : web micro log
tags : [python, tableau]
tagline: A Weekend Hack
---



This weekend, I decided to create a visualisation, to the best of my ability using the hat restaurant list from [Gault and Millau Sydney Restaurant Guide](http://www.noodlies.com/2013/11/2014-gaultmillau-sydney-restaurant-guide/).

The result is [here](http://public.tableausoftware.com/views/HatRestaurantVisualisation/HatRestaurants-Sydney?:embed=y&:display_count=no), and the screenshot is below:

![init-repo](/img/sydney-hat/Hat_Restaurants_-_Sydney.png)




---

## Lessons Learn

### Scraping the Table

This is not the first (and probably not the last time) I will scrap data off websites like that. Firstly the table from [www.noodlies.com](http://www.noodlies.com/2013/11/2014-gaultmillau-sydney-restaurant-guide/) is slightly
malformed. It isn't quite in the format you expect, and is not in unicode format which makes apostrophes look
strange in python (when you force it into unicode). 

Also, the list of restaurants aren't "table rows", but rather its placed using the `<p>` tag. This is fine, and isn't really a big deal.

The harder issue I had was getting the "hats" for the restaurants. I couldn't manage to seperate 5 and 4 hats (since I read them row-wise) and decided it was too difficult (not worth my effort for a short hack) to implement it thoroughly.

### Scraping the Restaurants

Scraping the restaurant information made use of [http://www.urbanspoon.com/](http://www.urbanspoon.com/). 
I noted that to get the Sydney information you could just search using `urllib` with the query string
`http://www.urbanspoon.com/s/70?q=`. This worked fine when urbanspoon wasn't confused about the restaurant, but could easily be rectified when in the search page, by "clicking" on the first link using the [Python Beautiful Soup library](http://www.crummy.com/software/BeautifulSoup/).

The end result was something like this:

    q = r"http://www.urbanspoon.com/s/70?q="

    f = urllib.urlopen(q+rest)
    soup = BeautifulSoup(f)
   
    try:   
        address = get_addr(soup)
    except:
        link=r"http://www.urbanspoon.com/"+get_rest(soup)
        f = urllib.urlopen(link)
        soup = BeautifulSoup(f)
        address = get_addr(soup)

This allowed us to easily grab the address information so we can figure out the longitude and latitude information to plot it using Tableau. At this stage there were already parsing issues with restaurants not found. Since "most" of them were found using this method, I decided it was good enough. Perhaps it is something I will pursue in the future using other restaurant aggregation websites.

### Parsing and Scraping Address Information

The address information was far easier to handle. I simply used [openstreetmap.org](http://www.openstreetmap.org) [api](http://wiki.openstreetmap.org/wiki/Nominatim) to grab the OSM data in an XML format to gather the longitude latitude information.

	def get_lonlat(address):
		map_add = r"http://nominatim.openstreetmap.org/search?q=%s&format=xml&polygon=1&addressdetails=1"
		f = urllib.urlopen(map_add % (re.sub('[ ]+','+', address)))
		soup = BeautifulSoup(f)
		place = soup.find("place")
		return place

Again there were addresses which couldn't be parsed correctly using this method, and hence were ignored (though minor cleaning was done to remove "Shop 1, 10 Sydney St" and level variations, since that had no effect on longitude and lattitude).

### Concluding Remarks

Python's Beautiful Soup is an amazing library which can do so much. I think the results were far better than I expected, and perhaps in the future I will revisit it to get a better hit rate.

See this [gist for the code](https://gist.github.com/charliec443/7856223). _Note_: I would be lying if I claimed this code resembled something production ready.







