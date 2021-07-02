---
layout: post
category : web micro log
tags :
---

[Make files](https://en.wikipedia.org/wiki/Make_(software)) have been around for a long time. They are probably one of the most popular build automation tools being used. In this post, we will look at a "lighter" version that is implemented in Python called [`doit`](http://pydoit.org/).

Since the goal is not necessarily to have a complicated build tool, but rather automate simple tasks, `doit` is perfect for this situation. In an analytics situation, the goal of this automation tool generally is:

*  Run a particular workflow (for example, a daily run)
*  Run a particular workflow on an "off" day (for example, an adhoc run)
*  Run any tests that are required. This could include:
   *  Integration tests  
   *  Unit tests  

Simple Usage
------------

The basic usage for `doit` is using the commandline argument `doit`. By default it considers the configuration within the file `dodo.py`. 

The simplest usage is running command line arguments. This can be created in two ways:

1.  Defining task using `task_*` pattern.   
2.  Using `yield` with a dictionary with the information. 

In the first pattern, the resulting task is is given the task name determined by the `*`. For example, 

```py
def task_tests():
    return {'actions': [cmd]}
```

Will create a task called `tests` which can be run via 

```sh
doit tests
```

In the second example using yield, we can create the task name based on the `basename` key. For example, 

```py
def gen_pipeline():
    yield {'basename': 'run pipeline',
           'actions': ['echo runpipeline'], 
           'verbosity': 2}
    yield {'basename': 'run test', 
           'actions': ['echo runtest'], 
           'verbosity': 2}

def task_all():
    yield gen_pipeline()
```

Will create two tasks, which can be run via:

*  `doit "run pipeline"`
*  `doit "run test"`

### Setting the default task

By default, `doit` with no parameters will run every single task. To restrict this behaviour, we can set this behaviour via the variable `DOIT_CONFIG`. For example having `DOIT_CONFIG = {'default_tasks': ['test']}` will make it run only the task `test` when `doit` is used from the commandline. 

Adding Commandline Arguments
----------------------------

Commandline arguments are added via string interpolation. 

```py
def task_cmd_params():
    # doit cmd_params -f 1 -p 2
    return {'actions':["echo mycmd %(flag)s %(param1)s"],
            'params':[{'name':'flag',
                       'short':'f',
                       'long': 'flag',
                       'default': '',
                       'help': 'helpful message about this flag'}, 
                       {'name': 'param1', 
                        'short': 'p', 
                        'default': ''}],
            'verbosity': 2
            }
```

This is fairly simply and can allow quite complex workflows.





