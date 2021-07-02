---
layout: post
category : 
tags : 
tagline: 
---

<!-- progressive model validation using metrics with hoeffding bounds is in the onlineml river items
so that we can adapt models and measure them in an online setting. -->

In Python there is [Gooey](https://github.com/chriskiehl/Gooey) which generates a GUI over a commandline program. How might we do this in Nim?

There are a few challenges with this. For example, from the standard library in Nim, there is no way to automatically assign and create types to arguments. Instead the standard library only supports:

*  flags
*  key value pairs
*  argument input

which assumes everything is also a string argument. 

To start off with, lets make some simplifying assumptions:

*  we will only support key-value pairs
*  we will store everything in a table (i.e. a hash table)
*  the program will only take in a table as input

With that we can start constructing our program. 


```nim
# these are the fields we generate
type optNames = enum
  hello, world

# no need to touch these, these hold the values of 
# various things
type 
  argEntry = array[optNames, Entry]
var optEntry: argEntry
var optTable = initTable[string, string]()
```

We'll start with defining what are the `keys` which we want, defined as `optName`. We'll also define what is ran; this is what is actually ran.

```nim
proc run() =
  for k,v in optTable:
    echo k, ":", v
```

So then we can create something with the argparser which will run correctly:

```nim
proc cmdlineMain*() = 
  # usage: nim c -r <file.nim> --hello:123 --world=what
  var p = initOptParser(commandLineParams())
  while true:
    p.next()
    case p.kind
    of cmdShortOption, cmdLongOption:
      optTable[p.key] = p.val
    else:
      break
  # run the program off optTable
  run()

cmdlineMain()
```

Then we can create something similar with `nim/ui`.

```nim
proc forceQuit(w: Window) = 
  destroy(w)
  system.quit()

proc runQuit(w: Window) = 
  # run the program first and then quit
  for n in optNames:
    optTable[$n] = optEntry[n].text
  run()
  forceQuit(w)


proc guiMain*() =
  # usage:
  var mainwin: Window
  var group: Group
  let mainBox = newVerticalBox(true)
  
  mainwin = newWindow("Hello World", 640, 480, true)
  mainwin.margined = true
  mainwin.onClosing = (proc (): bool = return true)
  mainwin.setChild(mainBox)
  
  # generate the fields from optName
  for n in optNames:
    group = newGroup($n)
    mainBox.add(group, false)
    optEntry[n] = newEntry("")
    group.child = optEntry[n]

  group = newGroup("")
  mainBox.add(group, false)

  let confirmationBox = newHorizontalBox(true)
  group.child = confirmationBox
  confirmationBox.add newButton("OK", proc() = runQuit(mainwin))
  confirmationBox.add newButton("Cancel", proc() = forceQuit(mainwin))
  show(mainwin)

init()
guiMain()
mainLoop()
```

Next time maybe we'll look at extending it to flags and other items.



