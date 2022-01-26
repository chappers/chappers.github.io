---
layout: post
category: web micro log
tags: [python]
---

Over the Christmas break I went on a holiday with a group of friends,
and we stumbled on a bit of a dilemma:

Who owes what?

Of course its easy to determine that before your trip, with the plane
tickets and accommodation. But what about during your trip? The taxi fares,
the meals, souvenirs, etc. You wouldn't want to pick it through with a fine
comb whilst trying to _enjoy_ your holiday!

As with these kind of problems we used spreadsheet, and I'll share here
the logic that we did using [Pandas](http://pandas.pydata.org/). Now in hindsight
doing this using Pandas may not be the wisest way. However with the requirements
I had in mind, it was by far the most straight forward solution I could think of.

Requirements:

- Information presented in a flat file (in this case csv) for easy manipulation
  on a phone or tablet.
- Should be "obvious" for a general user to stare at the flat file and understand
  the format, and how to populate new entries.

Again, this wasn't designed to be used by your average Joe, more for myself and perhaps
if other people for some inexplicable reason are interested in this. As a side note, if
I was to implement this in a production setting, the library would probably look similar
to [Advanced Python Scheduler](http://pythonhosted.org/APScheduler/); though this would
violate the requirements I set for myself.

## Data Format

Since I demanded that the data be a flat file, its quite simple.

    description, amount, creditor, debitor
    deposit, 38, Chapman, Tim
    breakfast, 10, Dave, Chapman
    Taxi, 20, Josh
    breakfast, 25, Josh
    Sea Kayaking, 250, Chapman
    ATV tour, 350, Chapman
    Taxi, 5, Josh, Tim
    Groceries, 23.5, Dave

Where the `creditor` would be the person to paid for the item, and `debitor` the person who owes money. I've placed it so that if `debitor` field is blank, then everyone owes money (and is to be split evenly). In future, I would increase the debitor field to accept any number of people.

## Massaging the Data

Next would be completely fill in the table to have full information. Since when `debitor` is omitted it means everyone else is required to be a debitor, and that the amount is split
amongst everyone. The secret here is to get a list of all persons and use `permutations`
from the module `itertools` to "fill in" the DataFrame using an outer join/merge.

{% highlight python %}
s = info.creditor.append(info.debitor).dropna()
person = list(set(s))
pls = pd.DataFrame(list(permutations(person,2)), columns=['creditor','debitor'])

infonan = info[-info['debitor'].isin(person)].drop('debitor', 1)
infonan['amount'] = infonan['amount']/len(person) #split the amounts evenly
ppt = info[info['debitor'].isin(person)].append(pd.merge(infonan, pls, how='outer', on=['creditor']))
{% endhighlight %}

But we're not done yet! After this is done, we would want a final DataFrame that consists
of the aggregated amounts of what each person owes to everyone. This is done using a
self-join. In SQL-like code it would be:

    select
        coalesce(a.creditor, b.debitor) as creditor,
        coalesce(b.creditor, a.debitor) as debitor,
        coalesce(a.amount, 0) as cr,
        coalesce(b.amount, 0) as dr
    from
        data as a
        full outer join data as b
            on a.creditor = b.debitor and a.debitor = b.creditor

Which would result in something like this:

<table>
<thead>
<tr>
<th></th>
<th>creditor</th>
<th>debitor</th>
<th>cr</th>
<th>dr</th>
</tr>
</thead>
<tbody>
<tr>
<th>0</th>
<td> Chapman</td>
<td>    Dave</td>
<td> 150.000</td>
<td>  15.875</td>
</tr>
<tr>
<th>1</th>
<td> Chapman</td>
<td>    Josh</td>
<td> 150.000</td>
<td>  11.250</td>
</tr>
<tr>
<th>2</th>
<td> Chapman</td>
<td>     Tim</td>
<td> 188.000</td>
<td>   0.000</td>
</tr>
<tr>
<th>3</th>
<td>    Dave</td>
<td> Chapman</td>
<td>  15.875</td>
<td> 150.000</td>
</tr>
<tr>
<th>4</th>
<td>    Dave</td>
<td>    Josh</td>
<td>   5.875</td>
<td>  11.250</td>
</tr>
</tbody>
</table>

## Determining Who Owes What

Finally how do we determine who owes what? Well the pseudo code is like this:

    p = largest creditor (value)
    Do for every row
        If the creditor or debitor is not p then
            subtract the creditor's amount from amount owing to p
            add the debitor's amount to the amount owing to p
        Else
            Remove any debits amounts from p's credit amounts
        End If
    End Do

The ending result would look like this:

<table>
<thead>
<tr>
<th></th>
<th>creditor</th>
<th>debitor</th>
<th>cr</th>
</tr>
</thead>
<tbody>
<tr>
<th>0</th>
<td> Chapman</td>
<td> Dave</td>
<td> 134.625</td>
</tr>
<tr>
<th>1</th>
<td> Chapman</td>
<td> Josh</td>
<td> 160.375</td>
</tr>
<tr>
<th>2</th>
<td> Chapman</td>
<td>  Tim</td>
<td> 165.875</td>
</tr>
</tbody>
</table>

And thats it, you've figured out who owes what.

[Notebook](http://nbviewer.ipython.org/gist/chappers/8253724/who-owes-what.ipynb) and [gist](https://gist.github.com/chappers/8253724)
