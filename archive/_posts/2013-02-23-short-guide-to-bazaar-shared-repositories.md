---
layout: post
category: web micro log
tags: [bazaar]
tagline:
---

This guide is meant as an introduction to shared repositories in [Bazaar](http://bazaar.canonical.com/en/). I am by no means an expert at Bazaar, hence there is no guarentee that all terms provided are correct.

## Shared Repositories

Whenever we want to share our repositories across different users, it may be wise to use _shared repositores_.

To create a shared repository:

    $ bzr init-repo foo-repo

`init-repo` : Creates a shared repository (this is the same command as `init-repository`
`foo-repo` : this is the name of the repository

**Bazaar Explorer**

Under Bazaar Explorer, I like to use the "All" button:

![init-repo](/img/bazaar-guide/init-repo.png)

### Adding Branches

To add branches (e.g. stable, development branches) we use:

    $ bzr init bar-repo

`init` : Makes a directory into a version branch
`bar-repo` : this is the name of the branch

**Bazaar Explorer**

Under Bazaar Explorer, I like to use the "All" button, making sure you navigate to the repository as created above:

![init](/img/bazaar-guide/init-repo.png)

---

Now you can simply drag and drop files in the same manner that you normally would.

So then, what's the difference? Why would we bother?

Version control allows us to keep track of all changes. Whenever you wish to preserve a state you must **Commit** the changes.

**Commit**ted changes will also have the _author_'s name and comments attached to it, to ensure that we understand _why_ a commit was needed for a production piece of code and to provide an audit log if necessary.

This is where I found `Bazaar Explorer` to be friendly than other GUI and open source versions of version control such as:

- [msysgit](http://code.google.com/p/msysgit/)
- Tortoise, [SVN](http://tortoisesvn.tigris.org/), [CVS](http://www.tortoisecvs.org/), [Bzr](http://wiki.bazaar.canonical.com/TortoiseBzr), [Git](http://code.google.com/p/tortoisegit/)

Although this may be seen as 'amateur' by most programmers, the truth is for data analysts whose programming consists of recording macros, and writing SAS code, and pre-generated SQL, this is a god-send. In my opinion, `Bazaar Explorer` does an amazing job.

The difference between different version control suites are negligible, since data analysts would place more focus on ease of use rather than features. To data analysts the differences would be more cosmetic than anything else, since scripts are not complex, nor would our knowledge be well developed.

## Add

You must first add files to be monitored by `Bazaar`. Simply follow on-screen instructions and this will be completed for you.

**Bazaar Explorer**

Click on the `+` sign under "What's Next":

![add](/img/bazaar-guide/add.png)

## Commit

The advantage of `Bazaar` compared with `SVN`, `Git`, `CVS` is the ease-of-use with its GUI under Windows.

To **commit** changes to a branch, navigate to the repository, then the branch, and then commit the changes. (You can view changes between the current version and the previous version using **Diff**).

**Bazaar Explorer**

Click on the 'arrow' sign under "What's Next" and type in your message:

![add](/img/bazaar-guide/add.png)

## Pull/Push

**Pull** new revisions from another branch making this branch a mirror of it. I like to think of it as getting the latest update from production branch.

**Push** new revisions to another branch making it a mirror of this one. I like to think of it as updating the production branch after we're satisfied with our development

**Bazaar Explorer**

1.  Click on the 'Pull/Push' icon in the toolbox under the toolbar
2.  Navigate to the branch you push to **Pull/Push** from

![pull](/img/bazaar-guide/pull.png)

## Merge

**Merge** changes from another branch into this one. Sometimes we might only be changing a single script, not everything in production. We would open our production repository and **Merge** changes to our production branch if thats what we desire.

**Bazaar Explorer**

1.  Click on the 'Merge' icon in the toolbox under the toolbar
2.  Navigate to the branch you push to **Merge** from

![merge](/img/bazaar-guide/merge.png)

The nice thing about merges and logs is the nice summary you would have

![log](/img/bazaar-guide/log.png)
