---
layout: post
category : proofs
tags : [proofs]
tagline: Possibly horrible measures...
---

In the area of machine learning, and statistical modelling, logistic regression, and the use of grouping objects in groups is extremely important.

There are plenty of documented ways to access model suitability, primarily dealing with false positive and true positive ratios. But when items are difficult (or expensive) to access and the prevailence of type III errors (correct classification for the wrong reasons, which strangely enough is quite important in some areas), different methods have to be employed to think about model fitness.

# Motivation

On key area which may be of importance is experimental design, particularly contigency tables. We have been thinking about how you could blend certain models together when the effects are not mutually exclusive.

Starting with the simplest measure of of performance we can look at correlation.

<table>
<thead>
<tr>
<th></th>
<th>y=1</th>
<th>y=0</th>
<th>total</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<th>x=1</th>
<td>\(p_{11}\)</td>
<td>\(p_{10}\)</td>
<td>\(p_{1 \bullet}\)</td>
</tr>
<tr>
<th>x=0</th>
<td>\(p_{01}\)</td>
<td>\(p_{00}\)</td>
<td>\(p_{0 \bullet}\)</td>
</tr>
<tr>
<th>total</th>
<td>\(p_{\bullet 1}\)</td>
<td>\(p_{\bullet 0}\)</td>
<td>\(p_{\bullet \bullet}\)</td>
</tr>
</tbody>
</table>

Since the the Pearson Correlation Coefficient reduces to the phi coefficient for the 2x2 contingency table, the we can represent

$$ \phi = \frac{p_{11} p_{00} - p_{10} p_{01}}{\sqrt{p_{1 \bullet}p_{0 \bullet}p_{\bullet 0}p_{\bullet 1}}} $$

But how does this actually perform in practise? Quoting wikipedia:

>  The phi coefficient has a maximum  value that is determined by the 
>  distribution of the two variables

And its not too hard to see. Consider the example below (adapted from "Phi/Phimax: Review and Synthesis", Davenport and El-Sanhurry, Educational and Psychological Measurement, 1991, 51) :

<table>
<thead>
<tr>
<th></th>
<th>y=1</th>
<th>y=0</th>
<th>total</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<th>x=1</th>
<td>0.38</td>
<td>0.42</td>
<td>0.8</td>
</tr>
<tr>
<th>x=0</th>
<td>0.02</td>
<td>0.18</td>
<td>0.2</td>
</tr>
<tr>
<th>total</th>
<td>0.4</td>
<td>0.6</td>
<td>1</td>
</tr>
</tbody>
</table>

For ease of understanding, let \\(N=100\\) so the contingency table would look like

<table>
<thead>
<tr>
<th></th>
<th>y=1</th>
<th>y=0</th>
<th>total</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<th>x=1</th>
<td>38</td>
<td>42</td>
<td>80</td>
</tr>
<tr>
<th>x=0</th>
<td>2</td>
<td>18</td>
<td>20</td>
</tr>
<tr>
<th>total</th>
<td>40</td>
<td>60</td>
<td>100</td>
</tr>
</tbody>
</table>

Then phi would look like

$$ \phi = 0.31 $$.

So if we change the cell value of x=0, y in (0,1), (i.e. changing whether group x belongs to group y or not), to measure the level of association, we can see that the max amount is highly reliant on the distribution. For the example above the maximum value of phi is 0.41.

if we tweak the values of \\(p_{01}, p_{00}\\) and look at the resulting values for phi (i.e. changing the marginal distribution), we observe:

<table>
<thead>
<tr>
<th>x=0, y=1</th>
<th>x=0, y=0</th>
<th>\(\phi\)</th>
<th></th>
</tr>
</thead>
<tbody>
<tr>
<td>8</td>
<td>12</td>
<td>0.00</td>
<td></td>
</tr>
<tr>
<td>7</td>
<td>13</td>
<td>0.05</td>
<td></td>
</tr>
<tr>
<td>6</td>
<td>14</td>
<td>0.10</td>
<td></td>
</tr>
<tr>
<td>5</td>
<td>15</td>
<td>0.15</td>
<td></td>
</tr>
<tr>
<td>4</td>
<td>16</td>
<td>0.20</td>
<td></td>
</tr>
<tr>
<td>3</td>
<td>17</td>
<td>0.26</td>
<td></td>
</tr>
<tr>
<td>2</td>
<td>18</td>
<td>0.30</td>
<td></td>
</tr>
<tr>
<td>1</td>
<td>19</td>
<td>0.36</td>
<td></td>
</tr>
<tr>
<td>0</td>
<td>20</td>
<td>0.41</td>
<td></td>
</tr>
</tbody>
</table>

# \\(\phi_{max}\\)

This leads to phimax. Since there is clearly a maximum value which phi can read (as we can see above, not necessarily 1) perhaps under certain conditions, the appropriate standardization would be to use \\(\frac{\phi}{\phi_{max}}\\). 

But firstly, how do we get the equation for \\(\phi_{max}\\), for \\(p_{\bullet 0} \ge p_{0 \bullet}\\)

$$ \phi_{max} = \frac{\sqrt{p_{1 \bullet} p_{\bullet 0}}}{\sqrt{p_{0 \bullet}p_{\bullet 1}}} $$

---

Since we are dealing with binary variables, then let

$$ X \sim Bern(p_x) $$
$$ Y \sim Bern(p_y) $$

So then the variance for \\(X, Y\\), we have

$$ s^2_x = p_x q_x $$
$$ s^2_y = p_y q_y $$

where \\( q_i = 1-p_i \\) for \\(i = x,y\\). Then the correlation is 

$$ \rho_{XY} = \frac{p_{xy} - p_x p_y}{s_x s_y} $$

where \\( p_{xy} \\) is the probability of \\(p_x\\) and \\(p_y\\) both happening.

### Maximizing \\(\phi\\)

Lets suppose that \\( p_x \gt p_y \\). Lets suppose that if \\(Y\\) occurs, then \\(X\\) also occurs, i.e. \\( p_{xy} = p_y \\).

Noting that this is the maximum, then 

$$ p_{xy} - p_x p_y = p_y - p_x p_y  = p_y q_x $$ 

substituting in to the formula above yields the required formulation of \\( \phi_max \\).

$$\begin{aligned}
\rho_{XY} & = \frac{p_{xy} - p_x p_y}{s_x s_y} \\\\
 & \le \frac{p_y q_x}{\sqrt{p_x q_x p_y q_y}} \\\\
  & = \frac{\sqrt{p_y q_x}}{\sqrt{p_x q_y }} \\\\
\end{aligned}$$

recognizing that \\( p_{1 \bullet}, p_{0 \bullet}, p_{\bullet 1}, p_{\bullet 0} \\) are equivalent to \\( p_x, q_x, p_y, q_y \\), we can further simplify the above

$$ \frac{\sqrt{p_y q_x}}{\sqrt{p_x q_y }}  = \frac{\sqrt{p_{\bullet 1} p_{0 \bullet}}}{\sqrt{p_{1 \bullet} p_{\bullet 0} }} $$ 



# \\(\frac{\phi}{\phi_{max}}\\) as a measure of Association

Indeed, if we use the same example as above, using \\(\frac{\phi}{\phi_{max}}\\) is indeed a simple adjustment. Applying it to the same contingency table as above:

<table>
<thead>
<tr>
<th>x=0, y=1</th>
<th>x=0, y=0</th>
<th>\(\phi\)</th>
<th>\(\frac{\phi}{\phi_{max}}\)</th>
</tr>
</thead>
<tbody>
<tr>
<td>8</td>
<td>12</td>
<td>0.00</td>
<td>0.13</td>
</tr>
<tr>
<td>7</td>
<td>13</td>
<td>0.05</td>
<td>0.25</td>
</tr>
<tr>
<td>6</td>
<td>14</td>
<td>0.10</td>
<td>0.50</td>
</tr>
<tr>
<td>5</td>
<td>15</td>
<td>0.15</td>
<td>0.38</td>
</tr>
<tr>
<td>4</td>
<td>16</td>
<td>0.20</td>
<td>0.50</td>
</tr>
<tr>
<td>3</td>
<td>17</td>
<td>0.26</td>
<td>0.63</td>
</tr>
<tr>
<td>2</td>
<td>18</td>
<td>0.31</td>
<td>0.75</td>
</tr>
<tr>
<td>1</td>
<td>19</td>
<td>0.36</td>
<td>0.88</td>
</tr>
<tr>
<td>0</td>
<td>20</td>
<td>0.41</td>
<td>1.00</td>
</tr>
</tbody>
</table>

We can now see that \\(\frac{\phi}{\phi_{max}}\\) has been scaled appropriately.

But we have to be slightly cautious when we use this. Consider that \\(\frac{\phi}{\phi_{max}}\\) can be simplified: 

$$\begin{align}
\frac{\phi}{\phi_{max}}  & =  \frac{p_{11} p_{00} - p_{10} p_{01}}{\sqrt{p_{1 \bullet}p_{0 \bullet}p_{\bullet 0}p_{\bullet 1}}} \div \frac{\sqrt{p_{\bullet 1} p_{0 \bullet}}}{\sqrt{p_{1 \bullet} p_{\bullet 0} }} \\\\\\\\
& = \frac{p_{11} p_{00} - (p_{1 \bullet}-p_{11})(p_{\bullet 1}-p_{11})}{p_{\bullet 1}p_{0 \bullet}} \\\\\\\\
& = \frac{p_{11} p_{00} - (p_{1 \bullet} p_{\bullet 1} - p_{11} (p_{1 \bullet} + p_{\bullet 1}) + p_{11}^2)} {p_{\bullet 1}(1-p_{1 \bullet})} \\\\\\\\
& = \frac{p_{11} (p_{00}+p_{1 \bullet} + p_{\bullet 1}-p_{11}) - p_{1 \bullet} p_{\bullet 1}} {p_{\bullet 1}-p_{\bullet 1} p_{1 \bullet}} \\\\\\\\
& = \frac{p_{11} (p_{00}+p_{11}+p_{10} + p_{11} + p_{01} - p_{11}) - p_{1 \bullet} p_{\bullet 1}} {p_{\bullet 1}-p_{1 \bullet} p_{\bullet 1}} \\\\\\\\
& = \frac{p_{11} - p_{1 \bullet} p_{\bullet 1}} {p_{\bullet 1}-p_{1 \bullet} p_{\bullet 1}} \\\\\\\\
\end{align} $$

This can be easily shown to be equivalent to

$$ \frac{p_{11} - p_{1 \bullet} p_{\bullet 1}} {p_{\bullet 1}-p_{1 \bullet} p_{\bullet 1}} = \frac{p_{00} - p_{0 \bullet} p_{\bullet 0}} {p_{0 \bullet}-p_{0 \bullet} p_{\bullet 0}} $$

There are practical concerns with the unity for phi / phimax. These mainly lying on the interpretation point of view. It highly depends on whether you believe that a maximum score of '1' should be given when all _possible_ agreements occur, this is _not_ the same as _total_ agreement. The issue is when the number of all possible is extremely small (with respect to the expected value. Remember if \\(p_{11} = p_{1 \bullet}p_{\bullet 1} \\) i.e. \\(p_{11}\\) is the expected value, phi and phi/phimax will be 0). 

phi / phimax is asymmetrical because it relies on the expected value of \\(p_{11}\\). So in the table above, we can move up 8 steps from the expected value, but down 12 from the expected value, compared with phi which is symmetrical. 

Note that when the marginal distributions are similar, phi and phi/ phimax will be similar, but when they are vastly differently, then the issue is that the range of phi becomes extremely restrictive and thus phi/phimax should be more useful. Although again, perhaps sensitive to small changes in sampling error. 

So we can see that phi/phimax is a reasonable measure of association, though care must be taken when using it. Unlike phi, phi/phimax is nonrobust, due to its sensitivity.
