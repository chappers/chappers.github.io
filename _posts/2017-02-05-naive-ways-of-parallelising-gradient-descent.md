---
layout: post
category : web micro log
tags :
---

It is easy to see that stochastic gradient descent is somewhat an "online learning" algorithm. It can be computed in a minibatch manner that can be parallised. If we take this idea to the extreme, we actually have an algorithm called the ["Hogwild!" algorithm](https://people.eecs.berkeley.edu/~brecht/papers/hogwildTR.pdf). The gist of this algorithm is:

```
Loop
  For some observation, with current solution "x":
    grad_v <- calculate gradient for only the "v"th column
    update "x_v" <- "x_v" - learning_rate * grad_v
  end
end
```

For a sufficiently large dataset with many columns, the chance of collision is small, and the worse case scenario that the algorithm updates twice is rather minimal.

In a language like Julia, this approach is easily implemented using `@sync @parallel`. In my own tests on my own computer, we could only see improvements on extremely large vectors; though the parallel implementation was also computationally less intensive on the memory side.

In this example, we simply wish to find the minimum point of the function \\(f(x) = x^2\\) where \\(x\\) is some extremely large vector (a bit of hand waving on the notation, but you get the idea).

```jl
if length(workers()) < 3
    addprocs(2)
end
workers()

@everywhere using Distributions

input = SharedArray(Float64, (100000))
input[:] = 10;

function hogwild_sgd(SA, dfunc, alpha, steps)
    size = length(SA)
    @sync @parallel for i=1:steps
        pos_rand = rand(1:size, 1)[1]
        SA[pos_rand] += -alpha * dfunc(SA)[pos_rand]
    end
    SA
end

@time hogwild_sgd(input, single_update, 0.05, 1000000)
# 204.839810 seconds (353.94 k allocations: 15.714 MB)

# comparison with single code.
function hogwild_sgd_single(SA, dfunc, alpha, steps)
    size = length(SA)
    for i=1:steps
        pos_rand = rand(1:size, 1)[1]
        SA[pos_rand] += -alpha * dfunc(SA)[pos_rand]
    end
    SA
end

input_single = ones(input)*10; # this is now an array, not SharedArray

@time hogwild_sgd_single(input_single, single_update, 0.05, 1000000)
# 278.976475 seconds (3.00 M allocations: 745.222 GB, 19.29% gc time)

```
