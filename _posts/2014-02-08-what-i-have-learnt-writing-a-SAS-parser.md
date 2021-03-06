---
layout: post
category: web micro log
tags: [python, sas]
---

So I've just released `v0.01-alpha` of my SAS transcompiler to Python ([Stan](https://github.com/chappers/Stan)).

Here is just a list of things I've learnt :

- SAS very similar to a PL\\0 language
- `by` statements are inferior to the `split-apply-combine` strategy
- `pyparsing` makes life very easy (compared with dealing with lots of regex)
- iPython magics are ridiculously easy to write
- writing Python packages isn't that hard, but there is **a lot** of extraneous options

Some of the (many) things which are missing:

- Just about every `proc` you can think of ... you can define your own as a "function". I know strictly speaking they are **not** the same thing, but for now it will do. (`proc sql` coming next release)
- As stated above no `by` statements, and hence none of the related statements as well (like `retain`).
- `if-else-then-do` not implemented correctly

But of course you would want to see it in action. So here it is!

    from stan.transcompile import transcompile
    import stan_magic
    from pandas import DataFrame
    import numpy as np
    import pkgutil


    import stan.proc_functions as proc_func

    mod_name = ["from stan.proc_functions import %s" % name for _, name, _ in pkgutil.iter_modules(proc_func.__path__)]
    exec("\n".join(mod_name))


    # create an example data frame
    df = DataFrame(np.random.randn(10, 5), columns = ['a','b','c','d','e'])
    df

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-1.402090</td>
      <td> 1.007808</td>
      <td>-0.761436</td>
      <td> 1.520951</td>
      <td>-0.287097</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-1.522315</td>
      <td>-0.170775</td>
      <td> 0.832071</td>
      <td>-0.640475</td>
      <td> 0.434856</td>
    </tr>
    <tr>
      <th>2</th>
      <td> 0.161613</td>
      <td> 1.753123</td>
      <td>-0.554494</td>
      <td>-0.102087</td>
      <td>-0.350737</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.797706</td>
      <td>-1.204808</td>
      <td>-0.405977</td>
      <td> 0.421891</td>
      <td>-0.347111</td>
    </tr>
    <tr>
      <th>4</th>
      <td> 0.287852</td>
      <td>-0.647063</td>
      <td> 1.323138</td>
      <td> 0.347085</td>
      <td> 0.606421</td>
    </tr>
    <tr>
      <th>5</th>
      <td> 1.711382</td>
      <td> 0.988707</td>
      <td>-0.287785</td>
      <td> 0.862959</td>
      <td> 0.981112</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.145970</td>
      <td>-0.030930</td>
      <td> 1.219454</td>
      <td>-0.544475</td>
      <td> 2.013656</td>
    </tr>
    <tr>
      <th>7</th>
      <td> 0.203527</td>
      <td>-0.460113</td>
      <td> 0.683482</td>
      <td>-1.917130</td>
      <td> 0.683844</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-0.397550</td>
      <td> 1.471630</td>
      <td> 0.826813</td>
      <td> 0.107800</td>
      <td> 0.094163</td>
    </tr>
    <tr>
      <th>9</th>
      <td> 0.012285</td>
      <td>-0.293033</td>
      <td>-0.133107</td>
      <td> 0.748343</td>
      <td> 0.290751</td>
    </tr>
  </tbody>
</table>
</div>

    %%stan
    data test;
    set df (drop = a);
    run;




    u"test=df.drop(['a'],1)\n"




    exec(_)
    test

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>x</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> 1.007808</td>
      <td>-0.761436</td>
      <td> 1.520951</td>
      <td>-0.287097</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-0.170775</td>
      <td> 0.832071</td>
      <td>-0.640475</td>
      <td> 0.434856</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>2</th>
      <td> 1.753123</td>
      <td>-0.554494</td>
      <td>-0.102087</td>
      <td>-0.350737</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-1.204808</td>
      <td>-0.405977</td>
      <td> 0.421891</td>
      <td>-0.347111</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-0.647063</td>
      <td> 1.323138</td>
      <td> 0.347085</td>
      <td> 0.606421</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>5</th>
      <td> 0.988707</td>
      <td>-0.287785</td>
      <td> 0.862959</td>
      <td> 0.981112</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.030930</td>
      <td> 1.219454</td>
      <td>-0.544475</td>
      <td> 2.013656</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-0.460113</td>
      <td> 0.683482</td>
      <td>-1.917130</td>
      <td> 0.683844</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>8</th>
      <td> 1.471630</td>
      <td> 0.826813</td>
      <td> 0.107800</td>
      <td> 0.094163</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-0.293033</td>
      <td>-0.133107</td>
      <td> 0.748343</td>
      <td> 0.290751</td>
      <td> 0</td>
    </tr>
  </tbody>
</table>
</div>

`if` statements combined with `do` `end` statements were difficult to implement.
Here is my current
implementation of if-then-else control flow, (I'll have to revisit `if` and `do`
`end` statements in the future...)

    %%stan
    data df_if;
        set df;
        x = if b < 0.3 then 0 else if b < 0.6 then 1 else 2;
    run;




    u"df_if=df\ndf_if['x']=df_if.apply(lambda x : 0 if x[u'b']<0.3 else 1 if x[u'b']<0.6 else 2  , axis=1)\n"




    exec(_)
    df_if

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
      <th>c</th>
      <th>d</th>
      <th>e</th>
      <th>x</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-1.402090</td>
      <td> 1.007808</td>
      <td>-0.761436</td>
      <td> 1.520951</td>
      <td>-0.287097</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-1.522315</td>
      <td>-0.170775</td>
      <td> 0.832071</td>
      <td>-0.640475</td>
      <td> 0.434856</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>2</th>
      <td> 0.161613</td>
      <td> 1.753123</td>
      <td>-0.554494</td>
      <td>-0.102087</td>
      <td>-0.350737</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-0.797706</td>
      <td>-1.204808</td>
      <td>-0.405977</td>
      <td> 0.421891</td>
      <td>-0.347111</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>4</th>
      <td> 0.287852</td>
      <td>-0.647063</td>
      <td> 1.323138</td>
      <td> 0.347085</td>
      <td> 0.606421</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>5</th>
      <td> 1.711382</td>
      <td> 0.988707</td>
      <td>-0.287785</td>
      <td> 0.862959</td>
      <td> 0.981112</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.145970</td>
      <td>-0.030930</td>
      <td> 1.219454</td>
      <td>-0.544475</td>
      <td> 2.013656</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>7</th>
      <td> 0.203527</td>
      <td>-0.460113</td>
      <td> 0.683482</td>
      <td>-1.917130</td>
      <td> 0.683844</td>
      <td> 0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>-0.397550</td>
      <td> 1.471630</td>
      <td> 0.826813</td>
      <td> 0.107800</td>
      <td> 0.094163</td>
      <td> 2</td>
    </tr>
    <tr>
      <th>9</th>
      <td> 0.012285</td>
      <td>-0.293033</td>
      <td>-0.133107</td>
      <td> 0.748343</td>
      <td> 0.290751</td>
      <td> 0</td>
    </tr>
  </tbody>
</table>
</div>

---

    # procs can be added manually they can be thought of as python functions
    # you can define your own, though I need to work on the parser
    # to get it "smooth"

    df1 = DataFrame({'a' : [1, 0, 1], 'b' : [0, 1, 1] }, dtype=bool)
    df1

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>a</th>
      <th>b</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>  True</td>
      <td> False</td>
    </tr>
    <tr>
      <th>1</th>
      <td> False</td>
      <td>  True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>  True</td>
      <td>  True</td>
    </tr>
  </tbody>
</table>
</div>

    %%stan
    proc describe data = df1 out = df2;
    by a;
    run;




    u"df2=describe.describe(data=df1,by='a')\n"




    exec(_)
    df2

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>a</th>
      <th>b</th>
    </tr>
    <tr>
      <th>a</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="8" valign="top">False</th>
      <th>count</th>
      <td>     1</td>
      <td>         1</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>     0</td>
      <td>         1</td>
    </tr>
    <tr>
      <th>std</th>
      <td>   NaN</td>
      <td>       NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td> False</td>
      <td>      True</td>
    </tr>
    <tr>
      <th>25%</th>
      <td> False</td>
      <td>      True</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>     0</td>
      <td>         1</td>
    </tr>
    <tr>
      <th>75%</th>
      <td> False</td>
      <td>      True</td>
    </tr>
    <tr>
      <th>max</th>
      <td> False</td>
      <td>      True</td>
    </tr>
    <tr>
      <th rowspan="8" valign="top">True </th>
      <th>count</th>
      <td>     2</td>
      <td>         2</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>     1</td>
      <td>       0.5</td>
    </tr>
    <tr>
      <th>std</th>
      <td>     0</td>
      <td> 0.7071068</td>
    </tr>
    <tr>
      <th>min</th>
      <td>  True</td>
      <td>     False</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>     1</td>
      <td>      0.25</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>     1</td>
      <td>       0.5</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>     1</td>
      <td>      0.75</td>
    </tr>
    <tr>
      <th>max</th>
      <td>  True</td>
      <td>      True</td>
    </tr>
  </tbody>
</table>
</div>

The proc actually isn't difficult to write. So for the above code it is actually
just this:

    def describe(data, by):
        return data.groupby(by).describe()

This functionality allow you to handle most of the `by` and `retain` cases. For
languages
like Python and R, the normal way to handle data is through the split-apply-
combine methodology.

Merges can be achieved in a similar way, by creating a `proc`:

    %%stan
    proc merge out = df2;
    dt_left left;
    dt_right right;
    on = 'key';
    run;




    u"df2=merge.merge(dt_left=left,dt_right=right,on='key')\n"




    left = DataFrame({'key': ['foo', 'foo'], 'lval': [1, 2]})
    right = DataFrame({'key': ['foo', 'foo'], 'rval': [4, 5]})

    exec(_)
    df2

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>key</th>
      <th>lval</th>
      <th>rval</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td> foo</td>
      <td> 1</td>
      <td> 4</td>
    </tr>
    <tr>
      <th>1</th>
      <td> foo</td>
      <td> 1</td>
      <td> 5</td>
    </tr>
    <tr>
      <th>2</th>
      <td> foo</td>
      <td> 2</td>
      <td> 4</td>
    </tr>
    <tr>
      <th>3</th>
      <td> foo</td>
      <td> 2</td>
      <td> 5</td>
    </tr>
  </tbody>
</table>
</div>

heres an example showing how you can define your own function and run it (not a
function
that came with the package)

    def sum_mean_by(data, by):
        return data.groupby(by).agg([np.sum, np.mean])


    %%stan
    proc sum_mean_by data = df_if out = df_sum;
    by x;
    run;




    u"df_sum=sum_mean_by(data=df_if,by='x')\n"




    exec(_)
    df_sum

<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="2" halign="left">a</th>
      <th colspan="2" halign="left">b</th>
      <th colspan="2" halign="left">c</th>
      <th colspan="2" halign="left">d</th>
      <th colspan="2" halign="left">e</th>
    </tr>
    <tr>
      <th></th>
      <th>sum</th>
      <th>mean</th>
      <th>sum</th>
      <th>mean</th>
      <th>sum</th>
      <th>mean</th>
      <th>sum</th>
      <th>mean</th>
      <th>sum</th>
      <th>mean</th>
    </tr>
    <tr>
      <th>x</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-1.962327</td>
      <td>-0.327055</td>
      <td>-2.806722</td>
      <td>-0.467787</td>
      <td> 3.519061</td>
      <td> 0.586510</td>
      <td>-1.584762</td>
      <td>-0.264127</td>
      <td> 3.682416</td>
      <td> 0.613736</td>
    </tr>
    <tr>
      <th>2</th>
      <td> 0.073355</td>
      <td> 0.018339</td>
      <td> 5.221268</td>
      <td> 1.305317</td>
      <td>-0.776902</td>
      <td>-0.194225</td>
      <td> 2.389623</td>
      <td> 0.597406</td>
      <td> 0.437441</td>
      <td> 0.109360</td>
    </tr>
  </tbody>
</table>
</div>
