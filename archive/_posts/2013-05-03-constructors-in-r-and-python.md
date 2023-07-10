---
layout: post
category: code dump
tags: [python, R]
tagline: A very quick comparison
---

The ability to construct your own abstractions is an important part of object orientated
programming language. In this post we will quickly run through the differences between
building constructors in `R` and `Python`.

# Declaring constructions

## Python

In python to declare a constructor we use:

    class Person:
      def __init__(self, name):
        self.name = name

In python methods can be added dynamically.

e.g.

from types import MethodType

    class Person:
    	def __init__(self, name):
    		self.name = name

    def add_age(self, age):
    	self.age = age
    	return None

    p = Person("chappers")
    p.add_age= MethodType(add_age, p)

    p.add_age(10)
    print p.age

Is totally valid code.

## R

Within R a all `fields` in a constructor must be declared and cannot be declared dynamically.
The R equivalent of the above is

    Person <- setRefClass("Person", fields = c("name", "age"))

    Person$methods(add_age = function(set_age) age <<- set_age)

    Chappers <- Person$new(name = "Chappers")
    Chappers$add_age(10)
    print(Chappers$age)
