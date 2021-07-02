---
layout: post
category : web micro log
tags : [j]
---

Today I thought I'll have a quick look at J and how to use J to solve some Euler project problems. This post will go through one possible way to solve the first Euler project problem.

> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23.
> Find the sum of all the multiples of 3 or 5 below 1000.

## How to read `J`

`J` is read right to left. For example:

```
  2+3*4
14
  3*4+2
18
```

We can see that "normal" order of operations is ignored! 

## Assigning things

We can assign things by using `=:` 

```
  x =: 1
  x
1
```

This is rather unspectacular. We can assign arrays in this fashion too:

```
  x =: 1 2 3 
  x
1 2 3
```

Remember since items are read from right to left, this means that we can assign and then do stuff to it.

```
  1 + x =: i. 10
1 2 3 4 5 6 7 8 9 10
  x
0 1 2 3 4 5 6 7 8 9
```

But remember, things are still ordered from right to left!

## Solving the question

Now that you have the basics of `J` I won't go into the other possible symbols, but I'll encourage you to read other tutorials and guides. Firstly we have get numbers that are divisible by 3 and 5.

First realise that the `modulo` operator is `|` (`%` is the division operator in `J`). So then if we apply the operator on the numbers 1-10 it would look like this:

```
  2|(1+i.10)
1 0 1 0 1 0 1 0 1 0
```

To actually select the numbers 1-10, we have to use the selection operator `#`

```
  (2|x)#x =: 1+i.10
1 3 5 7 9
```

Then to get the even numbers we can do something very similar

```
  (0=2|x)#x=:1+i.10
2 4 6 8 10
```

Combining it with the or operator `+.` and adding up the result we have the solution:

```
  +/((0=5|y)+.(0=3|y))#y=:i.1000
233168
```

## Comparing with other languages

Lets compare how we could solve the same problem in some other programming languages.

#### Imperative

**Python**

```py
ans = 0
for x in range(1000+1):
  if (x % 3) == 0 or (x % 5) == 0:
    ans += x
print ans
```

**Java**

```java
int ans = 0;
for(int i=0;i<=1000;i++){
    if ((i % 3 == 0) || (i % 5 == 0)) {
        ans += i;
    }
}
System.out.println(ans);
```

#### Functional

**Haskell**

```hs
sum $ filter(\n -> n `mod` 3 == 0 || n `mod` 5 == 0) [1..1000]
```

**Python**

```py
sum([x for x in range(1000+1) if (x % 5)== 0 or (x % 3) == 0])
```

#### Bonus

**R**

```r
sum(Filter(lambda(x ~ (x %% 3 == 0) || (x %% 5 == 0)), 1:1000))

# or the reverse!
require(magrittr)
1:1000 %>% Filter(l(x ~ (x %% 3 == 0) || (x %% 5 == 0)), .) %>% sum
```
