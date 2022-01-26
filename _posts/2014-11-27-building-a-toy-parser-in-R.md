---
layout: post
category: web micro log
tags:
---

In this blog post I will go through the process of building a combinatory parser in R. This post reflects
my intent of developing an R package called [Ramble](https://github.com/chappers/Ramble). Please note that at the time of writing
Ramble was still a work in progress and may differ to what I will explore in this blog post.

## R is a functional

What do I mean by this? In R functions are [first-class](http://en.wikipedia.org/wiki/First-class_function), and [higher-order](http://en.wikipedia.org/wiki/Higher-order_function).

```r
f <- function(...) {
  args <- list(...)
  return(args$fun)
}
```

In the example above, we could pass a function in the function `f`, or a variable. For example:

```r
> f(fun=function(x) x+1)
function(x) x+1
> f(fun=function(x) x+1) (1)
[1] 2
> f(fun=2)
[1] 2
```

Here we have an example of passing functions and variables around in functions. You can also see `?Map` or `?Reduce` for the other higher order functions in R.

## Building the Parser

When we build the parser, we want to build it up from the small parsers, eventually getting more complex. Lets start with, the easiest thing; taking the first character in the string.

```r
> string <- "STRING"
> list(result=substr(string, 1, 1), leftover=substring(string, 2))
$result
[1] "S"

$leftover
[1] "TRING"
```

The trick is we don't actually want to return the `list` when we parse something, we merely want to emit the function that does it. We'll follow this, by introduction the `item`, `returns`, `then` function:

```r
returns <- function(string) {
  return(function(nextString) {
    return(list(result = string, leftover=nextString))
  })
}

item <- function(...){
  return(function(string){
    return (if(string=="") list() else list(result=substr(string, 1, 1), leftover=substring(string, 2)))
  })
}

then <- function(parserp, parserf) {
  return(function(string) {
    result <- parserp (string)
    if (length(result) == 0) {
      return (list())
    }
    else {
      result_ <- parserf (result$leftover)
      return(list(result=c(result$result, result_$result), leftover=result_$leftover))
    }
  })
}
```

Combining them we get:

```r
> then(item(), returns("123")) ("abc")
$result
[1] "a"   "123"

$leftover
[1] "bc"
```

### Creating more complex results

Since the rest of the functions are notably more complex, I will ommit them here, but will discuss how they are used.

**Do**

The do function chains successive parsing actions together. If any of them fail, it will return an empty list. It has the last argument describing how/what it should do with these parsed elements.

**Choice**

The choice function tries the first parser, if it returns an empty list, it will move onto the second one.

These two functions can then be used to create things to create `many` and `many1` which are analogous to `*` and `+` in regex setting.

```r
many1 <- function(p) {
  do(v=p,
          vs=many(p),
          f = function(v,vs="") {unlist(c(v,vs))})

}

many <- function(p) {
  many1 (p) %+++% returns(list())
}
```

By now we can then go ahead and chain these functions again to determine things like natural numbers:

```r
nat <- function() {
  do(xs = many1(Digit()),f = function(xs) {paste(xs, collapse='')})
}

token <- function(p) {
  do(space(),
          v = p,
          space(),
          f = function(v) {v})
}

natural <- function(...) {token(nat())}

> natural() ("  123 abc")
$result
[1] "123"

$leftover
[1] "abc"
```

## Building an expression parser

We can now combine these parsers together to create a parser to handle complex expressions, taking into aspects like bracket and operation order. For this example, we will only consider `+`, `*` and brackets.

This grammar can be expressed in the following way:

```
expr = expr + expr | term
term = term * term | factor
factor = (expr) | nat
nat = 0 | 1 | 2 ...
```

This would be expressed as the following using Ramble:

```r
expr <- do(f=term,
           function(f, leftover_){
             return(
               (do(s=symbol("+"),
                   t=expr,
                   function(s,t) {
                     print(unlist(c(f,s,t)))
                     return(unlist(c(f,s,t)))
                   })
                %+++% return(f)) (leftover_)
             )
           })

term <- do(f=factor,
           function(f, leftover_){
             return(
               (do(s=symbol("*"),
                   t=term,
                   function(s,t) {
                     print(unlist(c(f,s,t)))
                     return(unlist(c(f,s,t)))
                   })
                %+++% return(f)) (leftover_)
             )
           })

factor <- (do(sl=symbol("("),
                   e=expr,
                   sr=symbol(")"),
              function(sl, e, sr) {
                print(unlist(c(sl, e, sr)))
                return(unlist(c(sl, e, sr)))
              })
           %+++% natural ())
```

The output will also reveal the how the expressions are parsed:

```
> factor("(1)")
[1] "(" "1" ")"
$result
[1] "(" "1" ")"

$leftover
[1] ""

> factor("1")
$result
[1] "1"

$leftover
[1] ""

> expr("1+(2*2)")
[1] "2" "*" "2"
[1] "(" "2" "*" "2" ")"
[1] "1" "+" "(" "2" "*" "2" ")"
$result
[1] "1" "+" "(" "2" "*" "2" ")"

$leftover
[1] ""

> expr("(1+1)*2")
[1] "1" "+" "1"
[1] "(" "1" "+" "1" ")"
[1] "(" "1" "+" "1" ")" "*" "2"
$result
[1] "(" "1" "+" "1" ")" "*" "2"

$leftover
[1] ""

> expr("1+2")
[1] "1" "+" "2"
$result
[1] "1" "+" "2"

$leftover
[1] ""

> term("1*1")
[1] "1" "*" "1"
$result
[1] "1" "*" "1"

$leftover
[1] ""
```

Hopefully in the future I can develop this further and make it easier to use. At the moment I feel that it is too clunky and not very friendly (although consistent with itself).
