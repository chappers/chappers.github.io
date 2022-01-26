---
layout: post
category:
tags:
tagline:
---

In this post I'll quickly go through my tips and tricks for using Colab in conjunction with my local environment.

## Setup

My general setup is to write a batch file locally and run it. Then change all references to local directories to the Google drive location. This involves adding a new cell:

```py
from google.colab import drive
drive.mount('/content/gdrive')
```

At the top of the file. To confirm this is working as intended you can verify by writing a dummy file like this:

```py
with open('/content/gdrive/My Drive/colab/hello.txt', 'w') as f:
  f.write("hello world")
```

So then to convert everything automatically, I simply used regex to find replace all patterns of a file to the new location of `/content/gdrive/My Drive/path/to/file`. In my case it was something similar to:

## Preparing the batch files

```py
import re

run_model = open('run_model.py', 'r').read()
run_model = re.sub("'folder/", "'/content/gdrive/My Drive/colab/folder/", run_model)

with open('run_model_colab.py', 'w') as f:
    f.write(run_model)
```

Once this is done, we can run the script in the manner you expect through:

```
%run run_model_colab.py
```

## "Coding" in Colab

Then the whole "notebook" would be:

```py
from google.colab import drive
drive.mount('/content/gdrive')

%run run_model_colab.py
```

## Future Considerations

Some of the things to think about is one could mount the files and sync with "My Drive" instead of continually uploading, as the environments are recycled every so often. This could be as easy as:

```py
from google.colab import drive
drive.mount('/content/gdrive')

!cp /content/gdrive/My Drive/path/to/py /content/myfile
```
