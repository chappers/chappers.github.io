---
layout: post
category : web micro log
tags : 
---

In a previous post, I looked at how I build Ramble and an example of a build a calculator. Since then, I have changed how the parser works and I thought it would be a good opportunity to demonstrate something you could do as a weekend hack in Ramble. Here I will take you through how to write a parser for a subset for a PL/0 language. I would probably need to spend a little more time to get the whole language working.

# Ramble library decisions

Compared with the previous post there are some functionality which have been remove or refactored. There has been many functions renamed to text variants, rather than the use of symbols. Those changes were rather cosmetic and will not be covered here. The two notable ones are below.

## Removal of `do` function

The `do` function was removed. This is because it was cumbersome and restrictive in the R language (though it definitely made sense in Haskell). The `do` function was also not required in implementing the expression parser, hence it only seemed to confuse people rather than assist in building readable parsers.

## Changing the `%then%` function

The `Unlist` function, which is very similar to the `R` base `unlist` function has been added to the `then` function.

```r
Unlist <- function(a.list) {
  hasLowerLevel = TRUE
  while(hasLowerLevel) {
    a.list1 <- unlist(a.list, recursive=FALSE, use.names=FALSE)
    if (is(a.list1, 'list')) {
      a.list <- a.list1
    }
    else {
      hasLowerLevel = FALSE
      return(a.list)
    }
  }
  warning("loop should have returned a list!")
  return(a.list)
}
```

How this function works is that it continues to unlist until the object is no longer a list. This function has **not** been optimised, and further work could be undertaken to improve it. The `then` function now looks like this:

```r
then <- function(p1, p2) {
  return(function(string) {
    result <- p1 (string)
    if (length(result) == 0) {
      return (list())
    }
    else {
      result_ <- p2 (result$leftover)
      if (length(result_$leftover) == 0 || is.null(result_$leftover)) {return(list())}
      return(list(result=Unlist(append(list(result$result), list(result_$result))), leftover=result_$leftover))
    }
  })
}
```

This allows  iterating over `$results` to be much easier when using the `%using%` function.

# What we are implementing

The [PL/0](http://en.wikipedia.org/wiki/PL/0) language is fairly simple. Wikipedia even has the model languaged defined in EBNF for us already:

```
program = block "." .

block = [ "const" ident "=" number {"," ident "=" number} ";"]
        [ "var" ident {"," ident} ";"]
        { "procedure" ident ";" block ";" } statement .

statement = [ ident ":=" expression | "call" ident 
              | "?" ident | "!" expression 
              | "begin" statement {";" statement } "end" 
              | "if" condition "then" statement 
              | "while" condition "do" statement ].

condition = "odd" expression |
            expression ("="|"#"|"<"|"<="|">"|">=") expression .

expression = [ "+"|"-"] term { ("+"|"-") term}.

term = factor {("*"|"/") factor}.

factor = ident | number | "(" expression ")".
```

In my attempt, I have implemented everything except `block`, `call` and `while` functionality.

Firstly, the expression grammar and lower has already been completed in my previous post, so we will only consider how `condition` and `statements` are implemented.

## condition

`condition` is implemented in a similar way, note that we can use `%using%` and then use `R` code to develop the actual condition.


```r
condition <- (expr %then% (token(String("<="))
                           %alt% token(String(">="))
                           %alt% symbol("=") 
                           %alt% symbol("<")
                           %alt% symbol(">")) 
                   %then% expr
                   %using% function(bool) {
                     if (bool[[2]] == "<") {
                       try(bool1 <- as.numeric(bool[[1]]) < as.numeric(bool[3]), silent=TRUE)
                       return(if (is.na(bool1)) bool else bool1)
                     }
                     else if (bool[[2]] == ">") {
                       try(bool1 <- as.numeric(bool[[1]]) > as.numeric(bool[3]), silent=TRUE)
                       return(if (is.na(bool1)) bool else bool1)
                     }
                     else if (bool[[2]] == "=") {
                       try(bool1 <- as.numeric(bool[[1]]) == as.numeric(bool[3]), silent=TRUE)
                       return(if (is.na(bool1)) bool else bool1)
                     }
                     else if (bool[[2]] == "<=") {
                       try(bool1 <- as.numeric(bool[[1]]) <= as.numeric(bool[3]), silent=TRUE)
                       return(if (is.na(bool1)) bool else bool1)
                     }
                     else if (bool[[2]] == ">=") {
                       try(bool1 <- as.numeric(bool[[1]]) >= as.numeric(bool[3]), silent=TRUE)
                       return(if (is.na(bool1)) bool else bool1)
                     }
                     else {
                       warning("boolean symbol should have matched, please check PL\\0 implementation")
                       return(bool)
                     }
                   })
```

We can use `try` here in a similar way to `try, except` in Python. Which is helpful if the expression cannot be evaluated. We can also make using of `warning` which can raise helpful information in the console when running the parser. 

## statement

The statement parser is created in much the same way. The only twist is the use of environment to `assign` and `get` the variables. We can use environments to "save" `procedures` once it gets implemented.

```r
statement <- (((identifier() %then% token(String(":=")) %then% expr)
               %using% function(stateVar) {
                 if (stateVar[[2]] == ":=") {
                   assign(stateVar[[1]], stateVar[[3]], env=PL0)
                 }
                 return(stateVar)
               })
            %alt% (symbol("!") %then% identifier() 
                    %using% function(stateVar) {
                      # this calls a defined function (procedure)
                      print(get(stateVar[[2]], envir = PL0))
                      return(stateVar)
                    })
            %alt% (token(String("if")) %then% condition %then% token(String("then")) 
                    %then% statement %using% function(x) {
                      if(x[[2]]) {
                        return(x[[4]])
                      }
                      else {
                        return(x)
                      }
                    })
             %alt% (token(String("begin")) %then% (statement %then% many(symbol(";") %then% statement))
                    %then% token(String("end")))
             # call not implemented
             # while loop not implemented
            )
```

From its usage we will be able to understand better how this actually works. 

For example, to assign 1 to `x`, increment `x` by 2, and print `x` we can write:

```r
> invisible(statement("begin x := 1; x := x+2; ! x end"))
[1] 3
```

