---
layout: post
category : 
tags : 
tagline: 
---

There is a certain appeal to building a machine learning pipeline once and deploying everywhere. Now often this refers to pipelines which are built via batch, and deployed as a batch, an api or via a stream; in this post, I thought I'll explore what it may mean if we build a Python pipeline once and deploy over node.

**Advantages**

*  Easier integration into the node ecosystem - we no longer have to worry about mixing languages, nor are we necessarily tied to a pure python server-side stack. Now of course this isn't _really_ true in the first place, but some of the complexity might now be removed
*  Speed to deployment - if tooling and build system is correct, we should be able to easily move things along directly into the node ecosystem.


**Disadvantages**

*  Transcompiling woes - how would you debug Python that generated Javascript code?
*  Performance - we would be 

**Approach**

The approach to do this can be found [here](https://github.com/charliec443/tfjs-python-pipelines). The general gist of it is that we are:

*  Using [transcrypt](http://transcrypt.org/) to convert plain Python objects to JavaScript
*  Using [tensorflow.js](https://js.tensorflow.org/) to handle things like converting TensorFlow models to JavaScript, allowing us to easily use these mature libraries in a sensible fashion. Afterall why would we want to rebuilt ML algorithms from scratch!

In a typically ML pipeline, you would want to

*  Perform some feature manipulation, e.g. convert text to a numeric vector
*  Convert categorical to one hot encoding

and many other things. In the Python ecosystem, we typically leverage [scikit-learn](http://scikit-learn.org/) to facilitate with this. However, as scikit-learn relies heavily on C bindings for performance reasons, this may prove difficult to port in an automated way. To resolve this, we may need to write a "pure python" re-implementation to get these benefits (maybe a mini-project for the future). 

Once these features are constructed, we can use them to train the model, and also ensure we can recover the parameters which have been learnt. Once this is complete, we can freeze it using `transcrypt`

```
python -m transcrypt -b -p .none -n $(py)
```

When working with node, we also may need to convert it (or use experimental modules); here we used `babel` to suit our needs

```
npx babel __target__/$(py).js --out-file $(py).js
```

Finally when we put it all together, we need to make sure everything can fit together. In this case, the pattern I used was to have a code template in the following manner:


```py
'use strict';
const tf = require('@tensorflow/tfjs')
require('@tensorflow/tfjs-node')

{{ my_pipeline }}

tf_predict = """async function predict(data){
    // Define a model.
    const model = await tf.loadModel('file:///path/to/model.json');
    model.predict(tf.tensor2d(data, [1, data[0].length])).print();
}
var pred = predict(X_transformed)
console.log(pred)
```

This approach has worked well so far for the use cases that I have



