---
layout: post
category : 
tags : 
tagline: 
---

When you work in data analytics realm, whether you are performing exploratory analysis or some production grade machine learning model, the workflow really shouldn't be all that different. 

This post is just a summary of what I do at this current point in time. In essence it is reduced to two broad ideas:

1.  Reduce context switching
2.  Make your stuff portable

## Reduce Context Switching

To reduce context switching means that one should aim to use the same set of tools no matter what they are doing. In general I've achieved this by only using one editor: VSCode. Currently this fulfils:

*  Exploratory analysis
*  Writing documentation and papers
*  Running experiments
*  Producing glue code

Perhaps what is better to understand, is what is VSCode actually replacing? VSCode replaces my need to have a separate LaTeX editor, it removes my need to have a separate environment for Python or R workflows (to a certain extent). 

What this means is that I don't use notebooks. Notebooks just aren't reproducible or test-able, and by extension detrimental in the long run when you need to provide reproducible code or require publishing items in a production grade environment. That being said, I still do use RStudio more than I'd admit, for the purposes of visualisation and other quick 10 line scripts which can be easily ran and launched. 

### Watchdog

Watchdog is a Python library which I use to orchestrate my LaTeX jobs. Currently, how this works is whenever a `*.tex` file is saved it launches off a job to build the appropriate LaTeX files for previewing. 

```py
import time
from watchdog.observers import Observer
from watchdog.events import PatternMatchingEventHandler

patterns = "*"
ignore_patterns = ""
ignore_directories = False
case_sensitive = True
my_event_handler = PatternMatchingEventHandler(patterns, ignore_patterns, ignore_directories, case_sensitive)

def runner(event):
    import subprocess 
    import os
    from sys import platform
    if ".tex" in event.src_path:
        print("Running...")
        print(platform)
        if platform.startswith('win'):
            os.system("runner.bat")
        else:
            os.system("./runner.sh")


my_event_handler.on_created = runner
my_event_handler.on_deleted = runner
my_event_handler.on_modified = runner
my_event_handler.on_moved = runner

path = "."
go_recursively = True
my_observer = Observer()
my_observer.schedule(my_event_handler, path, recursive=go_recursively)


my_observer.start()
try:
    while True:
        time.sleep(10)
except KeyboardInterrupt:
    my_observer.stop()
    my_observer.join()
```

Where the runners are in the form

```sh
pdflatex -interaction=nonstopmode main
bibtex  main
pdflatex -interaction=nonstopmode main
pdflatex -interaction=nonstopmode main
```

This in general gets the work done and more importantly is portable to different computers. 

## Make your stuff portable

My current hardware setup is mixed; this is simply due to the nature of computers and my own day job. In my day job, most of my work runs in AWS off Ubuntu or CentOS machines, where the underdying VDI is actually windows. At home, my computer runs Windows, but concurrently, my laptop is a MacBook. This combination is simply a combination of using things I'm used to, and in the scenario of the MacBook, it honestly is in my opinion the best portable device on the market right now. With a size smaller than the MacBook Air (ironically) and weighing in under 1kg, it really is a laptop which I can easily take to work, university and home which is why I use it. 

However, with these, comes certain challenges: enabling an environment and code which is reproducible. It is for this reason VSCode is used, as it can be installed on all operating systems and is a fully fledged IDE compared with Jupyter Lab. I have found that I did require some settings to be consistent across all my environments:

`settings.json`

```json
    "multiCommand.commands": [
        {
            "command": "multiCommand.runBlock",
            "sequence": [
                "editor.action.clipboardCopyAction",
                "workbench.action.terminal.paste"
            ]
        }
    ],
```

`keybindings.json`

```json
{ "key": "ctrl+enter", "command": "workbench.action.focusActiveEditorGroup", "when": "terminalFocus"},
{ "key": "ctrl+enter",  "command": "multiCommand.runBlock",
```

What this key allows me to do is to use `ipython` in an interactive way to easily move blocks of code to the console and back without too much hassle.

Getting to know your own editor is one of the best ways to save time and effort when continually on the move! Overall I've had success with this approach, but perhaps in the future I will revisit it when newer things come along.