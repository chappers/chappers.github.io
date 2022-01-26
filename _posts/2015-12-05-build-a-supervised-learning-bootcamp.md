---
layout: post
category: web micro log
tags:
---

Following my previous post, I had the idea pull together a series of modules which demonstrate how you could build supervised models from the bottom up. Through this you would have a strong appreciation of the underlying processes and models used in supervised learning. I think this is also a really good time to learn a bit of Julia.

```jl
abstract Learner

type LinearModel <: Learner
    coef

    function LinearModel(X, y)
        this = new()
        this.coef = \(X, y)
        this
    end
end

function predict(lrn::LinearModel, X)
    X*lrn.coef
end

#' Usage:
# a = [1 2;3 4]
# b = [1;0]
# linModel = LinearModel(a, b)
# predict(linModel, a)
```

Lets consider the code block above which implements a very simple linear regression model. Within Julia we can create an abstract data type which is akin to the "superclass" of all our supervised learner.

Building on top of this, a linear regression can be easily be solved through linear algebra as shown above. We can write a generic `predict` function which will work with all our learners through using multiple dispatch method.

We can follow this up with KNN:

```jl

type KNN <: Learner
    dataX
    dataY
    N

    function KNN(X, y, neighbours)
        this = new()
        this.dataX = X
        this.dataY = y
        this.N = neighbours
        this
    end
end

function predict(lrn::KNN, X)
    predX = zeros(size(X)[1])
    for ix in 1:size(X)[1]
        y_indx = sortperm(sum(abs2(lrn.dataX - repmat(X[ix, :], size(lrn.dataX)[1])), 2)[:], rev=false)
        predX[ix] = mean(lrn.dataY[[y_indx[1:lrn.N]]]) # select the first N entries
    end
    return predX
end
```

The interesting functions are more around `sortperm`, which I used inplace of `np.argsort`.

Things get harder when you start building regression trees. I attempted it but my Julia knowledge is rather lacking and it is reflected in my poor code decision, which just seems to be scrappily put together:

```jl
type RegressionTree <: Learner
    # this implementation focuses on variance spits instead of entropy or gini impurity
    # arrays are split as follows, where index...:
    # 1 : is the index of the parent node, 0 implies it is the root node
    # 2 : 1, if condition in parent node was satified, 0 if it wasn't
    # 3 : is the variable (by column index) that we should split on
    # 4 : is the value of the split. Will always assume that we are interested in predicate pattern value > ?
    decisionTree
    dataX
    dataY


    function RegressionTree(X, y, depth)
        function getSubset(decisionTree, X, y, leaf)
            # traverses through the decision tree to get the subset applicable to the
            # current branch
            if leaf == 1
                return X, y
            else
                parent   = decisionTree[leaf, 1]
                isParent = decisionTree[leaf, 2]
                col      = decisionTree[parent, 3]
                var      = decisionTree[parent, 4]

                if isParent == 1
                    subsetX = X[X[:, col] .> var, :]
                    subsety = y[X[:, col] .> var, :]
                else
                    subsetX = X[X[:, col] .< var, :]
                    subsety = y[X[:, col] .< var, :]
                end
                return getSubset(decisionTree, subsetX, subsety, parent)
            end
        end

        function getBestVariance(xs, y)
            # takes in the vector and response, and outputs the optimal value
            # to split on and the variance criteria amount.
            if size(xs)[1] == 0
                return [0, 0]
            end

            colTable = zeros(size(xs)[1], 2)
            for (row, x) in enumerate(xs)
                g1 = sum(abs2(xs[xs .> x] - y[xs .> x]))
                g2 = sum(abs2(xs[!(xs .> x)] - y[!(xs .> x)]))

                #g1 = 0 ? g1 == [] : g1
                #g2 = 0 ? g1 == [] : g1

                colTable[Int(row), 1] = g1 + g2
                colTable[Int(row), 2] = x
            end

            return colTable[[Int(sortperm(colTable[:, 1])[1])], :]
        end

        this = new()
        this.dataX = X
        this.dataY = y

        # size can be predetermined as described in comments above
        # we can also fill things out;

        # depth 2
        #0 1 0 0
        #1 1
        #1 0
        #2 1
        #2 0
        #3 1
        #3 0

        # depth 3
        #0 1 0 0
        #1 1
        #1 0
        #2 1
        #2 0
        #3 1
        #3 0
        #4
        #4
        #5
        #5
        #6
        #6
        #7
        #7

        this.decisionTree = zeros(2^depth-1, 4)
        this.decisionTree[2:size(this.decisionTree)[1], 1] = vcat({i * ones(2) for i in 1:(size(this.decisionTree)[1])/2}...)
        this.decisionTree[2:size(this.decisionTree)[1], 2] = repmat([1, 0], Int((size(this.decisionTree)[1]-1)/2))

        # now fill in the tree; for simplicity we will always fill to the full depth
        # in practise it isn't this simplistic.
        for branch in 1:size(this.decisionTree)[1]
            println(branch)
            subsetX, subsety = getSubset(this.decisionTree, X, y, branch)

            varTable = zeros(size(subsetX)[2], 3)
            varTable[:,1] = 1:size(subsetX)[2]

            # 1 : best split
            # 2 : variance number
            # calculate best split location.
            for column in 1:size(subsetX)[2]
                cols = subsetX[:, column]
                varTable[column, 2:3] = getBestVariance(cols, subsety)
            end

            bestSplit = varTable[[sortperm(varTable[:, 2])][1], :]
            this.decisionTree[branch, 3] = bestSplit[1]
            this.decisionTree[branch, 4] = bestSplit[3]
        end
        this
    end
end

```

Extending on this, we can further using bagging and boosting. This won't be shown here in Julia, but the idea in Python is somewhat straightforward. Simply resample the points based on the classification error; if the classification error is high, then increase the sampling probability.
