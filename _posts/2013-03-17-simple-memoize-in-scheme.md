---
layout: post
category : code dump
tags : [scheme]
tagline: 
---

Here is some simple code to memoize a recursive function in Scheme. I originally got this from a Stackoverflow question, but the link is now dead, so I thought I will post it up here.

**Memoize function:**   

	(define (memoize op)
	  (letrec ((get (lambda (key) (list #f)))
		(set (lambda (key item)
		  (let ((old-get get))
			(set! get (lambda (new-key)
		  (if (equal? key new-key) (cons #t item)
			(old-get new-key))))))))
		(lambda args
		  (let ((ans (get args)))
			(if (car ans) (cdr ans)
		  (let ((new-ans (apply op args)))
			(set args new-ans)
			   new-ans))))))

After we have memoized this, we can use this within our recursive function. For example, Challenge 121 (Intermediate)

>   _This problem uses the same money-changing device from Monday's Easy challenge._    
>    
>   Bytelandian Currency is made of coins with integers on them. There is a coin for each non-negative integer (including 0). You have access to a peculiar money changing machine. If you insert a N-valued coin, it pays back 3 coins of the value N/2,N/3 and N/4, rounded down. For example, if you insert a 19-valued coin, you get three coins worth 9, 6, and 4. If you insert a 2-valued coin, you get three coins worth 1, 0, and 0.   
>    This machine can potentially be used to make a profit. For instance, a 20-valued coin can be changed into three coins worth 10, 6, and 5, and 10+6+5 = 21. Through a series of exchanges, you're able to turn a 1000-valued coin into a set of coins with a total value of 1370.
>    
>    Starting with a single N-valued coin, what's the maximum value you could get using this machine? Be able to handle very large N.
>    
> _Author: Thomas1122_

Can be solved through memoization with the following code:

	(define (memoize op)
	  (letrec ((get (lambda (key) (list #f)))
		(set (lambda (key item)
		  (let ((old-get get))
			(set! get (lambda (new-key)
		  (if (equal? key new-key) (cons #t item)
			(old-get new-key))))))))
		(lambda args
		  (let ((ans (get args)))
			(if (car ans) (cdr ans)
		  (let ((new-ans (apply op args)))
			(set args new-ans)
			   new-ans))))))
	 
	(define coin-value (memoize (lambda (coin)
	  (if (insert? coin)
		(+ (coin-value (floor (/ coin 2)))
		   (coin-value (floor (/ coin 3)))
		   (coin-value (floor (/ coin 4))))
		coin))))
	 
	 
	(define (insert? coin)
	  (< coin 
	   (+ (floor (/ coin 2))
		  (floor (/ coin 3))
		  (floor (/ coin 4)))))
