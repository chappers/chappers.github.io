---
layout: post
category : web micro log
tags : 
---

Recently I have had time to implement a variation of the [RIDIT score](https://en.wikipedia.org/wiki/Ridit_scoring) in R. The variation used was in the form

$$
B_i = \sum_{j < i} P_j - \sum_{j > i} P_j
$$

Where \\(B_i\\) is the (transformed) RIDIT score for the value \\(i\\) and \\(P_j\\) is the probability of observing the value \\(j\\). This RIDIT score was proposed by Bross and used for Fraud Detection. The desirable features of this score is that it is in the range \\([-1, 1]\\), with a median value of \\(0\\). The score intuitively is very similar to the notion of a percentile.

Seeing that this algorithm is closely linked to the notion of percentile, we can simply use `ecdf` to calculate the proportions. My initial attempt (in pseudo-code) sounded something like this:

>  1. calculate all the percentiles of the numeric vector.  
>  2. lag the result.  
>  3. calculate the difference in the lag and non-lag would be the solution.  
>  4. to assign the scores, simply perform a lookup on each element of the table

Codified it would look like this:

```r
ridit_slow <- function(x) { 
  Fx <- ecdf(x) 
  ecdf.x <- environment(Fx)$x
  ecdf.y <- environment(Fx)$y
  
  df <- data.frame(names=ecdf.x, y=ecdf.y)
  df$y_lag <- c(0, df$y)[1:length(df$y)]
  df$ridit <- df$y_lag - (1-df$y)
    
  return(list(summary=df, ridit=df$ridit))
}

x <- rnorm(1000000)
scores_slow <- merge(data.frame(names=x), ridit_slow(x)$summary, all.x=TRUE)
```

The weakness of this is firstly the join and secondly the inability to interpolate data if we were to generate new scores. Also, several items would have to be copied every single time the code was run; if this could be done in minimal number of parses we could easily speed this code up.

The next approach I considered was simply following the code that was presented for the `stats::ecdf`

>  1. tally the first occurence of each element of the vector.  
>  2. calculate the cumulate sum for all values from smallest to largest and largest to smallest.  
>  3. Use approxfun to create a stepwise class for all our variables with the ridit scores. 

This approach definitely required greater thinking, and on first glance it doesn't appear to be equivalent to the actual definition of RIDIT. However for the vector based nature of R, this turns out to be a better improvement. In many ways this makes sense, as calculating and modifying the cumulative distribution will yield the RIDIT scores. 

```r
ridit_fast <- function(x) { 
  x <- sort(x)
  n <- length(x)
  vals <- unique(x)
  
  # calculates number less than val - number more than current val, will be normalised by n
  tabulate_x <- tabulate(match(x, vals))
  less_val <- cumsum(tabulate_x)
  more_val <- rev(cumsum(rev(tabulate_x)))
  
  rval <- approxfun(vals, (less_val - more_val)/n, 
                    method = "constant", yleft = -1, yright = 1, f = 0, ties = "ordered")
  class(rval) <- c("stepfun", class(rval))
  assign("nobs", n, envir=environment(rval))
  attr(rval, "call") <- sys.call()
  
  rval
}

scores_fast <- ridit_fast(x)(x)
```

Now we will have the ability to interpolate values (using approxfun, note that the option we have selected here is `f=0`) and do minimal number of copying numeric vectors. In essense, compared with the `ridit_slow`, the computational power of `ridit_fast` would have completed at the end of the very first statement.

Speed-wise we would notice roughly 3-4 times increase on my computer just by making these changes:

```r
ptm <- proc.time()
scores_slow <- merge(data.frame(names=x), ridit_slow(x)$summary, all.x=TRUE)
proc.time() - ptm
#   user  system elapsed 
#   3.75    0.05    3.82 

ptm <- proc.time()
scores_fast <- ridit_fast(x)(x)
proc.time() - ptm
#   user  system elapsed 
#   1.20    0.02    1.25 
```



