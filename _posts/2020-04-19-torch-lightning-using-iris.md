---
layout: post
category:
tags:
tagline:
---

Okay, so there are many articles on using torch with lightning and training with pytorch. But for whatever reason many of them are just overly complicated and talk through complicated workflows. For me, the details are important, but to start off, oftentimes we just want to know how do I do a "fit" and a "predict". In this post, we'll look briefly how to set up a minimal example for pytorch.

## Setting up the `pytorch` module

The pytorch module (will use this interchangeably with "lightning"), is the following methods:

- `forward` - which defines our network
- `train_dataloader` - which defines how we are batching/transforming our data/how is it loaded?
- `configure_optimizers` - which defines our optimizer
- `training_step` - which defines a single optimizer step

If you have all these defined as a module, then the training is as easy as:

```py
model = Model()
trainer = Trainer(max_epochs=1000)
trainer.fit(model)
```

And that's it! Onward to an iris example:

```py

import torch
from torch.nn import functional as F
from torch import nn
from torch.utils.data import TensorDataset, DataLoader
from torch.optim import Adam

import pytorch_lightning as pl
from pytorch_lightning import Trainer

import numpy as np
from sklearn.datasets import load_iris
from sklearn.preprocessing import OneHotEncoder
from sklearn.metrics import log_loss, accuracy_score


class Model(pl.LightningModule):
    def __init__(self):
        super().__init__()
        self.layer_1 = torch.nn.Linear(4, 5)
        self.layer_2 = torch.nn.Linear(5, 3)

    def forward(self, x):
        x = self.layer_1(x)
        x = torch.relu(x)
        x = self.layer_2(x)

        # proba_labels
        x = torch.log_softmax(x, dim=1)
        return x

    def train_dataloader(self):
        X, y = load_iris(return_X_y=True)
        iris_tensor = TensorDataset(torch.from_numpy(X).float(), torch.from_numpy(y))
        return DataLoader(iris_tensor, batch_size=64)

    def configure_optimizers(self):
        return Adam(self.parameters(), lr=1e-3)

    def training_step(self, batch, batch_idx):
        x, y = batch
        logits = self.forward(x)
        loss = F.nll_loss(logits, y)

        # add logging
        logs = {"loss": loss}
        return {"loss": loss, "log": logs}


# running this!
model = Model()
trainer = Trainer(max_epochs=1000)
trainer.fit(model)
```

We can finally verify that this works as expected:

```py
X, y = load_iris(return_X_y=True)
y_onehot = OneHotEncoder(sparse=False).fit_transform(y.reshape(-1, 1))

y_pred = torch.softmax(model(torch.from_numpy(X).float()), dim=1).detach().numpy()

# to verify this looks somewhat correct!
print(log_loss(y_onehot, y_pred))
print(accuracy_score(y, np.argmax(y_pred),))
```

Nice and easy!
