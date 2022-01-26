---
layout: post
category: web micro log
tags:
---

Testing is an important part of writing reusable code and having confidence when deployment production code. How should we write tests?

In many analytics applications, code is generally in the form of scripts, and there is rarely a need to completely "package" an application; especially when you are still prototyping code or in a "discovery" or "experiment" phase. In this post, I will look at how we can create simple templates in Python and R in order to be able to write and develop tests with confidence when we have simple scripts.

Within both languages we will write a simple `add_func` function which takes in two parameters and simply adds them together. With this function, we will also write a simple test to demonstrate how we can prepare ourselves for continuous integration.

## Python

Using the Python language, I will make use of the `nosetest` package. Firstly create a folder structure similar to below:

```
.
|   sample.py
|
\---tests
        sample_tests.py

```

Where the contents of `sample.py` is:

```py
def add_func(x,y):
    return x+y
```

`sample_tests.py`

```py
from nose.tools import *

from sample import add_func

def test_add():
    assert_equal(add_func(1,2), 3)
```

Within `nosetest`:

1.  Test files go into `tests/` and should be names `*_tests.py`.
2.  Within `*_tests.py`, all tests start with `test_*`

Running Python tests would be via:

```
nosetests
```

If the files are not picked up in `*.nix` then this might be because the tests are labeled as "executable". To examine this, run:

```sh
nosetests -vv --collect-only
```

If you wish to force nose to consider executables then run:

```sh
nosetests --exe
```

## R

Within R, I will make use of two libraries `testthat` and `import`. Import is used to "simulate" a script to a packge. The folder structure for R will be similar to Python:

```
.
|   sample.R
|
\---tests
        test_sample.R
```

Where the contents of `sample.R` is

```R
add_func <- function(x, y) {
  #' Add function
  #'
  #' @param x numeric number to be added
  #' @param y numeric number to be added
  #'
  #' @examples
  #' add_func(1,2) # 3
  return(x+y)
}
```

and `test_sample.R`:

```R
setwd("../")
import::from(sample.R, add_func)

test_that("simple add function works", {
  expect_equal(add_func(1,2), 3)
})
```

R running tests:

```sh
Rscript -e "library(testthat);test_dir(\"./tests\")"
```

In the code `test_sample.R` above, `import` is from the `import` library and `setwd("../")` will ensure that the relative directory location is picked up correctly from the `import` library.

Using `testthat`:

- All tests should go into `tests`
- All tests should have the pattern `test_*.R`
