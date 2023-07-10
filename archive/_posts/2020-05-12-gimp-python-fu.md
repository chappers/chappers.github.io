---
layout: post
category:
tags:
tagline:
---

**Bonus**

To make a document looked "scanned" - maybe we should add a `python-fu` script to do this as well!

```
convert -density 90 input.pdf -rotate 0.5 -attenuate 0.2 +noise Multiplicative -colorspace Gray output.pdf
```

---

I've recently been working on custom workflows using Gimp's `python-fu` plugins; I'm actually surprised I haven't come across it more in more image based workflows!

In this workflow, you can add layers and do calculations on an image, without manipulating and forcing pixel level changes you might be forced to on a pixel level. Furthermore, if you append text, you can modify the said text (rather than consider it on a pixel by pixel level).

There are plenty of tutorials and guides on how this works, so I won't really go into too much detail, but more on the "meta-programming" aspect of it.

### Gimp workflows

Gimp workflows can be thought of as inherently "batch based". Yes - there are manners of registering a set of procedures which can be run and then added to an image in an interactive way - but in most automation its batch.

My workflow follows this kind of pattern:

- Create a script with the function name as the script name
- Write the script in Gimp `python-fu` (keep in mind it is Python 2)
- Write execution script which can be multiple `python-fu` procedures (I've done it in Python 3)

This can become a bit of a mess down the track, but at least its helped me stay sane. The naming convention also helps reason with the code in separate procedures (however it isn't very good at "reusing" sets of procedures). This then ends up feeling like how you might code up blocks of procedures and trying to manage that - basically an absolute mess. So there should be better ways (though its bottlenecked by the "procedural database" approach in GIMP...)

### Thinking about the execution script

The execution script in this context does end up being a bit of a hack, essentially you end up with a dictionary of parameterised procedures to execute. For example here: https://github.com/chappers/gimp-python-experiments/blob/master/gimp-plugins/export_batch.py

We have code which looks like this:

```py
import subprocess

# see here: https://stackoverflow.com/questions/44430081/how-to-run-python-scripts-using-gimpfu-from-windows-command-line
template = {
    "jpg_to_xcf": """gimp-console -idf --batch-interpreter python-fu-eval -b "import sys;sys.path.insert(0, './gimp-plugins'); import jpg_to_xcf; jpg_to_xcf.jpg_to_xcf('{}',)" -b "pdb.gimp_quit(1)" """,
    "xcf_to_jpg": """gimp-console -idf --batch-interpreter python-fu-eval -b "import sys;sys.path.insert(0, './gimp-plugins'); import xcf_to_jpg; xcf_to_jpg.xcf_to_jpg('{}',)" -b "pdb.gimp_quit(1)" """,
    "xcf_to_jpg_showall": """gimp-console -idf --batch-interpreter python-fu-eval -b "import sys;sys.path.insert(0, './gimp-plugins'); import xcf_to_jpg_showall; xcf_to_jpg_showall.xcf_to_jpg_showall('{}',)" -b "pdb.gimp_quit(1)" """,
}
```

Which then gets executed via something like this:

```py
shcode = template["jpg_to_xcf"].format(f)
subprocess.call(shcode, shell=True)
```

This is "good enough" for this kind of code for now; and actually really powerful in workflows - though thinking about whether its extensible or maintainable is a different issue, as things do feel very "throwawayable".
