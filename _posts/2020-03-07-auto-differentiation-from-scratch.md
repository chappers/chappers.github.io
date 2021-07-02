---
layout: post
category : 
tags : 
tagline: 
---

In this post we'll look at building auto-differentiation from scratch in Python. Although there are many frameworks for doing this already, its always interesting to peak behind the covers and see how you might do it yourself!

The broad approach is to compose the variables and functions into components whereby the value and the gradients can be easily passed alongside each other. The most basic unit is a single variable

```py
class Variable(object):
    def __init__(self, value, grad=0):
        self.value = value
        self.grad = grad
        
    def __add__(self, v):
        value = self.value + v.value
        grad = self.grad + v.grad
        return Variable(value, grad)
    
    def __sub__(self, v):
        value = self.value - v.value
        grad = self.grad - v.grad
        return Variable(value, grad)
    
    def __mul__(self, v):
        value = self.value * v.value
        # chain rule!
        grad = self.grad * v.value + self.value * v.grad
        return Variable(value, grad)
    
    def __truediv__(self, v):
        value = self.value / v.value
        # quotient rule!
        grad = (self.grad * v.value - self.value * v.grad)/(v.value**2)
        return Variable(value, grad)
    
    def __pow__(self, v):
        value = self.value ** v.value
        assert v.grad == 0, "Gradient of powers not implemented"
        # y = f(x)^n --> y' = n f(x)^(n-1) * f(x)
        grad = v.value*(self.value ** (v.value - 1))*self.grad
        return Variable(value, grad)
```

We can observe how this works through a simple example. Let's say we wanted to calculate $3x$. Then to differentiate with respect to $x$ where the value of $x$ is $4$, we would write below

```py
eqn = Variable(3) * Variable(4, 1)
print("Gradient", eqn.grad) # print out 3
```

Regression
----------

Based on this we can easily construct a simple linear regression example. 

The key secret (and advantage) is that differentiation of the loss function can be provided "for free"!

```py
def mse(a, b):
    return (a-b)**Variable(2, 0)
```

This can easily calculate the gradient automatically when presented with items with the pattern: `mse(Variable, Variable)`. We just have to make sure that we differentiate with respect to the correct variable. 

To optimise each parameter we've simply computed separate gradients for each variable. In this example, we aim to use only one predictor variable in order to calculate $y = m x + b$. Using this as a simple spring board, we can see that in this scenario, we'll need to optimise 2 parameters. 

```py
import numpy as np

# generate some data
m = 3
b = -2

x = np.random.normal(size=(100,))
y = m*x + b + np.random.normal(size=(100,))/100
```

To calculate the gradients, the most naive way is to loop through the whole dataset multiple times and calculate all the gradients for all variables. 

```py
# let's find the line of best fit using auto differentiation!

m_prime = 0
b_prime = 0
lr = 0.1
iters = 1000

# gather all gradients
for _ in range(iters):
    m_grad = []
    b_grad = []
    for x_, y_ in zip(x, y):
        ym_pred = Variable(m_prime, 1) * Variable(x_) + Variable(b_prime)
        yb_pred = Variable(m_prime) * Variable(x_) + Variable(b_prime, 1)
        m_grad.append(mse(Variable(y_), ym_pred).grad)
        b_grad.append(mse(Variable(y_), yb_pred).grad)
    
    m_prime -= lr*np.mean(m_grad)
    b_prime -= lr*np.mean(b_grad)

print("m", m, "m prime", np.round(m_prime, 4))
print("b", b, "b prime", np.round(b_prime, 4))
```

```
# output
m 3 m prime 3.0001
b -2 b prime -1.9991
```

Using this we can calculate the MSE to demonstrate this works as expected

```py
from sklearn.metrics import mean_squared_error

predictions = []
for x_, y_ in zip(x, y):
    y_pred = Variable(m_prime, 0) * Variable(x_, 0) + Variable(b_prime, 0)
    predictions.append(y_pred.value)
print("MSE autodiff", mean_squared_error(y, predictions))


predictions = []
logloss_est = []
for x_, y_ in zip(x, y):
    y_pred = Variable(m, 0) * Variable(x_, 0) + Variable(b, 0)
    predictions.append(y_pred.value)
print("MSE truth", mean_squared_error(y, m*x + b))
```

```
# output
MSE autodiff 0.00010196434661564995
MSE truth 0.00010280944064901146
```

Classification
--------------

To expand this to classification, we'll need to define a few more primitive functions used in mathematics

```py
class Log(object):
    def __call__(self, v):
        value = np.log(v.value)
        grad = v.grad/v.value
        return Variable(value, grad)
    
log = Log()

class Exp(object):
    def __call__(self, v):
        value = np.exp(v.value)
        grad = np.exp(v.value) * v.grad
        return Variable(value, grad)
    
exp = Exp()

class Sigmoid(object):
    def __call__(self, v):
        value = 1/(1+np.exp(-v.value))
        grad = (value * (1-value))*v.grad
        return Variable(value, grad)

sigmoid = Sigmoid()

def logloss(true_val, pred_val):
    # avoid divide by 0 errors...
    pred_val = Variable(np.clip(pred_val.value, 1e-15, 1-1e-15), pred_val.grad)
    if true_val.value == 1:
        return Variable(-1, 0)*log(pred_val)
    else:
        neg_pred_val = Variable(1, 0) - pred_val
        return Variable(-1, 0)*log(neg_pred_val)
```

This is very much the same as constructing `mse` whereby we did not have to manually calculate or differentiate the equation and we "get it free" as part of the auto-differentiation framework.

From here we can proceed in exactly the same way as before

```py
# let's find the line of best fit using auto differentiation!

m_prime = 0
b_prime = 0
lr = 0.1
iters = 1000

# gather all gradients
for idx in range(iters):
    m_grad = []
    b_grad = []
    for x_, y_ in zip(x, y):
        ym_pred = Variable(m_prime, 1) * Variable(x_, 0) + Variable(b_prime, 0)
        yb_pred = Variable(m_prime, 0) * Variable(x_, 0) + Variable(b_prime, 1)

        ym_pred = sigmoid(ym_pred)
        yb_pred = sigmoid(yb_pred)

        m_grad.append(logloss(Variable(y_, 0), ym_pred).grad)
        b_grad.append(logloss(Variable(y_, 0), yb_pred).grad)
    
    m_prime -= lr*np.mean(m_grad)
    b_prime -= lr*np.mean(b_grad)

from sklearn.metrics import log_loss

predictions = []
logloss_est = []
for x_, y_ in zip(x, y):
    y_pred = Variable(m_prime, 0) * Variable(x_, 0) + Variable(b_prime, 0)
    predictions.append(sigmoid(y_pred).value)
    logloss_est.append(logloss(Variable(y_), y_pred).value)
print("Log loss autodiff", log_loss(y, predictions))


predictions = []
logloss_est = []
for x_, y_ in zip(x, y):
    y_pred = Variable(m, 0) * Variable(x_, 0) + Variable(b, 0)
    predictions.append(sigmoid(y_pred).value)
    logloss_est.append(logloss(Variable(y_), y_pred).value)
print("Log loss truth", log_loss(y, predictions))
```

```
# output
Log loss autodiff 0.10934973581815889
Log loss truth 0.1573882474767359
```

"Deep" Neural Network
---------------------

With all these pieces in place, we can construct a neural network; which is just stacking multiple layers like this - for now we'll ignore considerations in relation to non-linearities. This example is going to be tedious, because right now we are optimising one parameter at a time!

Lets make a neural network, whereby the first layer has size 2 and the second layer size 1 to make the prediction layer. This means there will be

4 parameters to optimise in the first layer, and 3 parameters to optimise in the 2nd layer (can you think of why?)

First we'll define our model; which is akin to defining your favourite TensorFlow or PyTorch library. Essentially the idea is that, we provide the inputs and target, as well as the weights and which weights it is to learn (via `learn_config`) this allows us to iterate over the model in a nice manner.

```py
def deep_model(x, y, param_config, learn_config={}):
    """
    m{layer}_{node} - the weights
    b{layer}_{node} - the intercept
    
    layer{layer}_{node}_out - the output of the {layer}
    """
    m0_0 = param_config.get("m0_0", 0)
    m0_1 = param_config.get("m0_1", 0)
    b0_0 = param_config.get("b0_0", 0)
    b0_1 = param_config.get("b0_1", 0)
    m1_0 = param_config.get("m1_0", 0)
    m1_1 = param_config.get("m1_1", 0)
    b1 = param_config.get("b1", 0)
    
    lw_m0_0 = learn_config.get("m0_0", 0)
    lw_m0_1 = learn_config.get("m0_1", 0)
    lw_b0_0 = learn_config.get("b0_0", 0)
    lw_b0_1 = learn_config.get("b0_1", 0)
    lw_m1_0 = learn_config.get("m1_0", 0)
    lw_m1_1 = learn_config.get("m1_1", 0)
    lw_b1 = learn_config.get("b1", 0)
    
    layer0_0_out = Variable(m0_0, lw_m0_0) * Variable(x) + Variable(b0_0, lw_b0_0)
    layer0_1_out = Variable(m0_1, lw_m0_1) * Variable(x) + Variable(b0_1, lw_b0_1)    
    layer1_0_out = layer0_0_out * Variable(m1_0, lw_m1_0) + layer0_1_out * Variable(m1_1, lw_m1_1) + Variable(b1, lw_b1)
    
    y_pred = sigmoid(layer1_0_out)
    return logloss(Variable(y), y_pred).value, logloss(Variable(y), y_pred).grad
```

Finally, we put it into a training loop just like what we did with the other examples:

```py
grad_info = {
    "m0_0":[], "m0_1":[], "b0_0":[], "b0_1":[], "m1_0":[], "m1_1":[], "b1":[]
}

grad_info_reset = grad_info.copy()

param_info = {
    "m0_0":np.random.normal(), "m0_1":np.random.normal(), "b0_0":np.random.normal(), "b0_1":np.random.normal(), "m1_0":np.random.normal(), "m1_1":np.random.normal(), "b1":np.random.normal()
}
param_list = list(grad_info.keys())
lr = 0.1
iters = 100

# gather all gradients
for idx in range(iters):
    grad_info = grad_info_reset.copy()
    train_log_loss = []
    for x_, y_ in zip(x, y):
        for param in param_list:
            grad_info[param].append(deep_model(x_, y_, param_info, {param:1})[1])
        # calculate logloss
        train_log_loss.append(deep_model(x_, y_, param_info)[0])
    
    for param in param_list:
        param_info[param] -= lr * np.mean(grad_info[param])
        
    if idx % 5 == 0 or idx == iters-1:
        print("Iter/Log loss: ", idx, np.mean(train_log_loss))
```

```
# output
Iter/Log loss:  0 1.5343089469987539
Iter/Log loss:  5 0.5314582040274584
Iter/Log loss:  10 0.2747447173557924
Iter/Log loss:  15 0.18498504745762442
Iter/Log loss:  20 0.1436244706549995
Iter/Log loss:  25 0.12188643807656602
Iter/Log loss:  30 0.10972878949590728
Iter/Log loss:  35 0.10265121666096096
Iter/Log loss:  40 0.09827646134365249
Iter/Log loss:  45 0.09522659386137193
Iter/Log loss:  50 0.09267849555751771
Iter/Log loss:  55 0.09015530096578099
Iter/Log loss:  60 0.08741069875057354
Iter/Log loss:  65 0.08435453269531379
Iter/Log loss:  70 0.0809997495824758
Iter/Log loss:  75 0.07742256951730055
Iter/Log loss:  80 0.07373233560154564
Iter/Log loss:  85 0.07004922542015078
Iter/Log loss:  90 0.0664885786253087
Iter/Log loss:  95 0.0631507328424087
Iter/Log loss:  99 0.06069513211906005
```


