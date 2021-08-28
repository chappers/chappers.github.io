---
layout: post
category : web micro log
tags :
---

To play around with [tensor flow I used vagrant](https://github.com/gavinln/tensorflow-ipy). The first thing that happend was a bug due to an older version of virtualbox; which didn't solve itself as I needed to upgrade my vagrant installation...

30 minutes later, `vagrant up` finally works, and another 30 minutes later everything is ready and installed.

![setup](/img/tf/setupvagrant.png)

We can then login via ssh:

```
Host: 127.0.0.1
Port: 2222
Username: vagrant
Private key: C:/Users/XXXXX/.vagrant.d/insecure_private_key
```

Now you can run through the tensorflow examples! I would recommend the [tutorials located here](https://github.com/nlintz/TensorFlow-Tutorials). Also check out the [tensoflow]() site to upgrade to the latest version. At the time of writing you could upgrade using:

    sudo pip install --upgrade https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.6.0-cp27-none-linux_x86_64.whl

We can give [`skflow` a try](https://github.com/google/skflow) which tries to mimic Scikit learn. To install this follow the instructions:

    sudo pip install git+git://github.com/google/skflow.git
    
Which should also install dependencies like `sklearn` (this might have to be done before installing `skflow`. This can be done using:

    sudo apt-get install build-essential python-dev python-numpy python-setuptools python-scipy libatlas-dev libatlas3-base    
    sudo apt-get install python-matplotlib    
    sudo apt-get install python-sklearn
    
We can create a simple classifier example using Iris dataset by following the examples on the website:

```py
import skflow
from sklearn import datasets, metrics
iris = datasets.load_iris()
classifier = skflow.TensorFlowClassifier(n_classes=3)
classifier.fit(iris.data, iris.target)
score = metrics.accuracy_score(classifier.predict(iris.data), iris.target)
print "Accuracy:", score
```
    
For some reason I couldn't get the deep neural net working; I will check it out a bit later! 


---

**Update**

I have set up a repository with my own vagrant instance. You can check it out here:

[https://github.com/chappers/skflow-tensorflow-vagrant](https://github.com/chappers/skflow-tensorflow-vagrant)
