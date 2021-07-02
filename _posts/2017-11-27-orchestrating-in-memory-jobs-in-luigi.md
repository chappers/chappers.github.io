---
layout: post
category : 
tags : 
tagline: 
---

Orchestrating in-memory jobs in Luigi need not be hard. The easiest way is to roll your own `luigi.Target`. For example the following works perfectly:

```py
_data = {}

class MemoryTarget(luigi.Target):
    _data = {}
    def __init__(self, path): 
        self.path = path
    def exists(self): 
        return self.path in _data
    def put(self, value): 
        _data[self.path] = value
    def get(self): 
        return _data[self.path]
```

The reason why I've kept the `_data` outside the function is simply to ensure that Python objects persist outside of their respective tasks (of course you could bring it into `MemoryTarget` if you didn't want that to happen). Then usage for this is quite straightforward.

Now you can put together tasks and use it to persist data and calculations performed by Luigi as needed:

```py
# my_module.py, available in your sys.path
import luigi


_data = {}

class MemoryTarget(luigi.Target):
    _data = {}
    def __init__(self, path): 
        self.path = path
    def exists(self): 
        return self.path in _data
    def put(self, value): 
        _data[self.path] = value
    def get(self): 
        return _data[self.path]


class MyTask(luigi.Task):
    x = luigi.IntParameter(default=5)
    y = luigi.IntParameter(default=45)
    
    #def run(self):
    #    print self.x + self.y
    
    def run(self):
        f = self.output()
        f.put(self.x + self.y)
        print(f.get())
    
    def output(self):
        return MemoryTarget(self.__class__.__name__)

class MyTask2(luigi.Task):
    x = luigi.IntParameter()
    y = luigi.IntParameter(default=45)
    
    def run(self):
        dyna_input = yield MyTask()
        f = self.output()
        f.put(dyna_input.get() + self.x + self.y)
        print(f.get())
    
    def output(self):
        return MemoryTarget(self.__class__.__name__)
    
    @luigi.Task.event_handler(luigi.Event.SUCCESS)
    def output_object(task):
        """Will be called directly after a successful execution
           of `run` on any Task subclass (i.e. all luigi Tasks)
        """
        return [1,2,3]

if __name__ == '__main__':
    luigi.run(["--local-scheduler", "--x", "15", "--y", "15"], main_task_cls=MyTask2)
    print(_data) # prints as expected.
```