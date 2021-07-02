---
layout: post
category : web micro log
tags : [python]
---

Recently I've been playing around with changing wallpapers, but I realised it is just so much work curating images to use. So I thought one way was to use Reddit and rely on various subreddits as inspiration.

As I got into it, I found that if the subreddit was relevant, I was just downloading everything. So why not automate this! Using Reddit's API, where all we have to do is add `.json` to the URL, we can easily scrape all the source URLs and download them. Here is the code:

```python
import json
import urllib2
import urllib

subreddit = 'EarthPorn'

response = urllib2.urlopen('http://www.reddit.com/r/%s/.json' % subreddit)
html=response.read()

raw_json = json.loads(html)
urllinks = [x['data']['url'] for x in raw_json['data']['children']]
i=0
print "There are %d links..." % len(urllinks)

for link in urllinks:
    try:
        urllib.urlretrieve(link, link.split('/')[-1])
        i = i+1
        print '%s : Success' % link
    except:
        print '%s : Failed!' % link
else:
    print "%.2f links succeeded" % (float(i)/float(len(urllinks)))
```

Now this code won't look into the URL if it is an album and download all the pictures within the album, however I think for the relative few lines of code, it has achieved quite a lot.

Perhaps this could be packaged into some kind of phone app, allowing you to have "random" wallpapers automatically downloaded based on your subreddit preferences. Now that could be an interesting next project.

### Update

**5 November 2014**

Added some tests to "try" to download `imgur` links.

```python
# get reddit and download all the pictures

import json
import urllib2
import urllib

subreddit = 'EarthPorn'

response = urllib2.urlopen('http://www.reddit.com/r/%s/.json' % subreddit)
html=response.read()

raw_json = json.loads(html)
urllinks = [x['data']['url'] for x in raw_json['data']['children']]
i=0
print "There are %d links..." % len(urllinks)

for link in urllinks:
    try:
        if link.endswith('.jpg') or link.endswith('.jpeg') or link.endswith('.png'):
          urllib.urlretrieve(link, link.split('/')[-1])
          i = i+1
          print '%s : Success' % link
        else:
          try:
            urllib.urlretrieve(link+'.jpg', link.split('/')[-1]+'.jpg')
            print '%s : Success' % link
          except:
            print '%s : Not downloaded' % link
    except:
        print '%s : Failed!' % link
else:
    print "%.2f links succeeded" % (float(i)/float(len(urllinks)))
```

