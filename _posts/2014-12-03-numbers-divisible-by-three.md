---
layout: post
category : web micro log
tags : 
---

Why is it that if a numbers is divisible by three, then the digits are also divisible by three? 

Consider every integer \\(x\\) can be written as 

$$ x = \sum_{i \in Z} a_i 10^i $$

where \\(Z\\) is the set of all integers, and \\(a_i \in {0, 1, ..., 9}\\) for all \\(i \in Z\\). This can be rewritten as

$$ x = \sum_{i \in Z} a_i (10^i-1+1) $$

It follows that if \\(10^i -1 \\) is divisible by three for all \\(i \in Z^+\\) then it follows that \\(x\\) is divisible by three if the sum of the digits are also divisible by three since:

$$ \sum_{i \in Z} a_i (10^i-1+1) \mod 3 = \sum_{i \in Z} a_i  \mod 3 $$

---

Lets begin by proving that \\(10^i -1\\) is divisible by three for all \\(i \in Z\\). 

If \\(i = 0\\), then we know that \\(0\\) is clearly divisible by three. We can also easily verify this is true for \\(i = 1\\) since \\(10-1 = 9\\) is clearly divisible by three. 

Now assuming that case \\(i=k\\) holds, we need to demonstrate that case \\(k+1\\) also hold. 

So assuming that 

$$ 10^k -1 \mod 3 = 0 $$ 

then

$$ 10^{k+1} - 1 \mod 3 = (10^{k} - 1 + 1) \times 10 - 1 \mod 3 $$

Since we have assumed that \\( 10^k -1 \mod 3 = 0 \\), the expression above simplifies to:

$$ 10-1 \mod 3 = 9 \mod 3 = 0 $$

as required. 

Therefore since 

$$ x = \sum_{i \in Z} a_i 10^i = \sum_{i \in Z} a_i (10^i-1+1) $$

Then 

$$ x = \sum_{i \in Z} a_i (10^i-1+1) \mod 3 = \sum_{i \in Z} a_i \mod 3 $$ 

as required. 


