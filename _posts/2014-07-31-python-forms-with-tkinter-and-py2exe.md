---
layout: post
category : web micro log
---

Decided to quickly pull together a simple application which could add entries to a sqlite database.

Some quick observations about `tkinter`:

*  `Entry` : is really easier to pick up input fields.  
*  `Label` : feels extremely hacky.  
*  `.grid()` : must be one of the ugliest things I've had to work with, but it works.  
*  Packaging up the app into a single file (i.e. `bundle_files` : 1) required some small changes to the `py2exe/build_exe.py` [due to a](http://stackoverflow.com/questions/14975018/creating-single-exe-using-py2exe-for-a-tkinter-program) [bug](http://sourceforge.net/p/py2exe/bugs/108/)

Other than that it was strangely quite painless. For the full repo see [here](https://github.com/charliec443/tkinter-forms).

