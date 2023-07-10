---
layout: post
category: web micro log
tags: [sas]
---

I've been debating whether `by` statement should be retained in [Stan](http://chappers.github.com/Stan). This is quite easily my least favourite "feature" of SAS. The modern methodology of "split-apply-combine" in R and Pandas is far superior and easier to read compared with the many hacks which come along. For example:

    data class;
        set sashelp.class;
        by sex; /*just pretend its been sorted...*/
        retain cumulative_age name_list;
        if first.sex then do;
            cumulative_age = 0;
            name_list = '';
        end;

        cumulative_age = cumulative_age + age;
        name_list = catx(', ', name);
        if last.sex then output;
    run;

is quite a common "recipe" in SAS coders who I have encountered. Often, it gets way more complex quite quickly. But under the "split-apply-combine", the formula is the same and consistent.

    df.groupby("sex").apply(<some function here>)

You don't have to worry about retain statements (I'm pretty sure my sas code above is actually
wrong; I can't avoid a SAS license to check) or whether you remembered the clumsy "first.", "last." idioms. Lets not get into running means and other weird custom data steps which can't
be solved via a `proc`.

Despite all of this, if Stan actually becomes popular someone may have to implement this.
This will be somewhere far down the to-do list!
