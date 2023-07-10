---
layout: post
category: web micro log
tags:
---

Today I am releasing [binst](https://github.com/chappers/binst) which an "optimal binning" package using supervised and unsupervised methods including:

- kmeans
- entropy
- decision trees

## Motivations

This package was firstly spurred by `smbinning` which to me seemed to be very confusing to use. This was what spurred the decision tree method in this package. Although this package "worked", I had trouble from an interoperability perspective to apply it on H2OFrames.

For example I wished to perform something as simple as:

```r
h2o_makecuts <- function(x, y) {
  # using discretization
  breaks <- c(min(as.vector(x)), max(as.vector(x)), cutPoints(as.vector(x), as.vector(y)))
  h2o.cut(x, breaks, include.lowest = T)
}
```

However this was very difficult using `smbinning` as it ended up looking like this:

```r

h2o_makecuts <- function(x, y) {
  # using the same approach as smbinning
  breaks <- make_ctreebreaks(as.vector(x), as.vector(y))
  h2o.cut(x, c(min(as.vector(x)), max(as.vector(x)), breaks), include.lowest = T)
}

make_ctreebreaks <- function(x, y) {
  df <- data.frame(x=x, y=y)
  build_tree <- ctree(y~x, df)
  breaks <- as.numeric(unique(unlist(list.rules.party(build_tree))))
  return(breaks)
}

list.rules.party <- function(x, i = NULL, ...) {
  #' stolen from source code
  if (is.null(i)) i <- nodeids(x, terminal = TRUE)
  if (length(i) > 1) {
    ret <- sapply(i, list.rules.party, x = x)
    names(ret) <- if (is.character(i)) i else names(x)[i]
    return(ret)
  }
  if (is.character(i) && !is.null(names(x)))
    i <- which(names(x) %in% i)
  stopifnot(length(i) == 1 & is.numeric(i))
  stopifnot(i <= length(x) & i >= 1)
  i <- as.integer(i)
  dat <- data_party(x, i)
  if (!is.null(x$fitted)) {
    findx <- which("(fitted)" == names(dat))[1]
    fit <- dat[,findx:ncol(dat), drop = FALSE]
    dat <- dat[,-(findx:ncol(dat)), drop = FALSE]
    if (ncol(dat) == 0)
      dat <- x$data
  } else {
    fit <- NULL
    dat <- x$data
  }

  rule <- c()

  recFun <- function(node) {
    if (id_node(node) == i) return(NULL)
    kid <- sapply(kids_node(node), id_node)
    whichkid <- max(which(kid <= i))
    split <- split_node(node)
    ivar <- varid_split(split)
    svar <- names(dat)[ivar]
    index <- index_split(split)
    if (is.factor(dat[, svar])) {
      slevels <- levels(dat[, svar])[index == whichkid]
      srule <- paste(svar, " %in% c(\"",
                     paste(slevels, collapse = "\", \"", sep = ""), "\")",
                     sep = "")
    } else {
      if (is.null(index)) index <- 1:length(kid)
      breaks <- cbind(c(-Inf, breaks_split(split)),
                      c(breaks_split(split), Inf))
      sbreak <- breaks[index == whichkid,]
      right <- right_split(split)
      srule <- c()
      if (is.finite(sbreak[1]))
        srule <- c(srule, sbreak[1])
      if (is.finite(sbreak[2]))
        srule <- c(srule, sbreak[2])
      #srule <- paste(srule, collapse = " & ")
    }
    rule <<- c(rule, srule)
    return(recFun(node[[whichkid]]))
  }
  node <- recFun(node_party(x))
  return(unlist(rule))
}

list.rules.party(test)
test <- make_ctreecut(iris[, 1], iris[, 5])

iris.hex <- as.h2o(iris, "iris.hex")
iris.hex$SL_Bin <- h2o_makecuts(iris.hex[, 1], iris.hex[, 5])
iris.hex$SL_Bin %>% as.vector() %>% unique

```

## Extensions

`kmeans` was mostly inspired by a similar approach I took to one dimensional clustering which was used in my CS 6601 course at Georgia Tech (note that in this course we had to implement our own variant of kmeans using Python, so you won't get any hints here!)

The approach using `entropy` is simply a wrapper to the `discretization` library within R.

## Future Work

If there is enough interest I hope to add other approaches to binning, probably using techniques such as

- Jenks Natural Breaks
- Support for categorical data
