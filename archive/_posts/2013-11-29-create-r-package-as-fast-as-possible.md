---
layout: post
category: code dump
tags: [R]
---

Apparently creating R packages is a good idea for code reuse. So what's the best way to do it?

Hopefully in this short blog post I will take you from start to finish as quickly as possible (omitting details for you to fill in)

# Useful Packages

Firstly install `devtools`:

`install.packages("devtools", dependencies = TRUE)`

This is for using `dev_mode()` which will isolate an environment for you to do more testing.

    library(devtools)
    dev_mode()

# create()

Navigate to the working directory and just create a package:

`create("mypackage")`

Put in your R scripts in the folder `R`. And that is basically it.
You have an R package finished! All that is left is to fill in documentation
and metadata for your package.

# Metadata

## DESCRIPTION

Fill in the `DESCRIPTION` file which should have been created when you can `create("mypackage")`.

# document()

Documentation is easily achieved through docstrings using the library `devtools` through the package `roxygen`.
Syntactically it is similar to markdown, this also is beyond the scope of this post.

If you wish to export this function in the package remember to use the docstring `@export`.

To build documentation simply use `document()`.

# test()

To add tests create a folder `inst/tests`, prepending all tests scripts with `test-*.r` where the
wildcard is well...a wildcard. To test, simply use `test()`.

# build()

Finally, build your package using `build()`, to create your `tar` file for sharing.

---

And thats it. You have built a package.

---

**Update**

[Here is a short screencast I did on creating R packages](https://www.youtube.com/watch?v=rmiCnQEnB3g).
