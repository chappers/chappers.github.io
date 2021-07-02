---
layout: post
category : code dump
tags : [python]
tagline: 
---

Scraping information from the internet can be very handy. But often, information is located somewhere within the tags of a webpage. 

Scraping this information into a format which can be used by another source can be easily achieved using [Beautiful Soup](http://www.crummy.com/software/BeautifulSoup/).

## An Example

In this example we shall retrieve the Barclay's premier table located [here](http://www.premierleague.com/en-gb/matchday/league-table.html).

To open this webpage we shall use `urllib2`.

	import urllib2

	webpage = urllib2.urlopen('http://www.premierleague.com/en-gb/matchday/league-table.html')

We can then "soup" up the webpage using `Beautiful Soup`.

	from bs4 import BeautifulSoup
	
	soup = BeautifulSoup(webpage)

To retrieve the information we shall use the `find` function in `Beautiful Soup`, to extract everything with the tag name of `table`.

	table = soup.find('table')

Finally to extract the information we would make use of the tags `tr` and `td` to extract the row and column information respectively.

	import re

	string = ''
	for row in table.find_all('tr'):
		for column in row.find_all('td'):
			string += "%s," % re.sub(r'\s+', ' ', column.get_text())
		string += '\n'

Which will provide the required table in the variable `string`.

And that's it!

---

The full code is as below:

	from bs4 import BeautifulSoup
	import urllib2
	import re

	webpage = urllib2.urlopen('http://www.premierleague.com/en-gb/matchday/league-table.html')
	soup = BeautifulSoup(webpage)

	# extract table
	table = soup.find('table')

	# now for each table you want to extract the row, then column
	# for each item in the column put a comma between it, and then if its
	# the next row then we add a new line

	string = ''
	for row in table.find_all('tr'):
		for column in row.find_all('td'):
			string += "%s," % re.sub(r'\s+', ' ', column.get_text())
		string += '\n'



