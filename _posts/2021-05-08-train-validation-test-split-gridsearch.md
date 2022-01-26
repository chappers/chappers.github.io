---
layout: post
category:
tags:
tagline:
---

In scikit-learn when you do a gridsearch the default option uses cross validation option. But what if we want to use a train-validation split instead?

This is preferable if we want to train models quicker, and without performing that kind of search procedure. This can be done through using `StratifiedShuffleSplit` explicitly in the `cv` option:

```py
GridSearchCV(
    estimator=GradientBoostingClassifier(**gbm_param),
    param_grid=param_grid,
    scoring=custom_score,
    n_jobs=4,
    cv=StratifiedShuffleSplit(
        n_splits=1, test_size=0.2, random_state=7
    ),  # faster, and with a validation split rather than full cv
    verbose=True,
)
```

Part of me thinks this approach should actually be the default option!

```py
from tpot import TPOTClassifier
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split, StratifiedShuffleSplit
import numpy as np

iris = load_iris()
X_train, X_test, y_train, y_test = train_test_split(iris.data.astype(np.float64),
    iris.target.astype(np.float64), train_size=0.75, test_size=0.25, random_state=42)

tpot = TPOTClassifier(generations=5, population_size=50, verbosity=3, random_state=42)
# 2 mins 55 secs
tpot.fit(X_train, y_train)

tpot = TPOTClassifier(
    generations=5,
    population_size=50,
    verbosity=3,
    cv=StratifiedShuffleSplit(
        n_splits=1, test_size=0.2, random_state=7
    ),
    random_state=42,
    log_file="test.log"
)
# 44 seconds
tpot.fit(X_train, y_train)
```

As we can see in this simple example, it is a much quicker alternative.
