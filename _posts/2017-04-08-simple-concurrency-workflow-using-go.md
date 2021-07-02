---
layout: post
category : web micro log
tags :
---

I've been thinking about designing my own concurrency workflow (or least thinking what it should look like). There are several tools which already exist, which if it was in a production setting I would probably use. As they say, what I can not build, I do not understand...

The idea is if we have shell scripts which run our code and is modular, then could we concurrently run our code? And once certain portions are done could it then know to combine the outputs together so that it could continue in some other workflow. 

This is to reflect the fact that I would want something that is flexible and have few dependencies. This is an example using only the standard library of `go`. 

The key idea of how you might do this in `go` is via `goroutines`. Essentially you write a function and stick `go` infront! Easy!

For example if I have functions `func1` and `func2`, to run them at the same time, we would simply do:

```go
go func1()
go func2()
```

The question then arises how do I know when they are finished? How do I share things that each one writes? In our example, since we will not always be in `go` land, we probably don't have to worry about sharing objects, as everything will be in `bash` script, and we are merely using `go` as an "enhanced" bash script, meaning the only thing we need to add is the `sync.WaitGroup`. The idea behind this is if we attach a some tasks to a group, then when the tasks are done, we can signal to the `WaitGroup` so that it will continue when it has noticed everything has finished. 

Below is a simple example of how this might look like:


```go
package main

import (
    "fmt"
    "os/exec"
    "sync"
    "log"
)

func runRScript(r_path string, wg *sync.WaitGroup) {
    fmt.Println("Running:", r_path)
    cmd, err := exec.Command("Rscript", r_path).Output()
    if err != nil {
        log.Fatal(err)
        fmt.Println(cmd)
    }
    fmt.Println("Done:", r_path)
    
    // lets the WaitGroup know it is now done and can continue
    wg.Done()
}

func main() {
    var wg sync.WaitGroup // you need waitgroup to know when the jobs are done to continue
    
    // adds the number of tasks it needs to wait for
    wg.Add(2)
    go runRScript("write_iris.R", &wg)
    go runRScript("write_iris2.R", &wg)
    
    // waits until the number of tasks in wg.Add(*) is done
    // will panic if it goes negative!
    wg.Wait()
    
    // once it is finished waiting it can continue    
    fmt.Println("Running combine...")
    exec.Command("Rscript", "combine_iris.R").Run()
    fmt.Println("Finished")
}
```

Where the R scripts are:

**`write_iris.R`**

```r
library(tidyverse)

Sys.sleep(5)
data(iris)
write_csv(iris, "iris.csv")


```

**`write_iris2.R`**

```r
library(tidyverse)

Sys.sleep(7)
data(iris)
iris$Sepal.Length <- iris$Sepal.Length+1
write_csv(iris, "iris2.csv")


```

**`combine_iris.R`**

```r
library(tidyverse)

iris1 <- read_csv("iris.csv")
iris2 <- read_csv("iris2.csv")

iris3 <- rbind(iris1, iris2)
write_csv(iris3, "iris3.csv")


```

