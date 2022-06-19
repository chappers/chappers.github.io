---
layout: post
category:
tags:
tagline:
---

I've started to do some more web development in Rust, and so far its been okay. I've managed to create something that is somewhat non-trivial and beginning to feel my way around the Rust ecosystem and thinking about machine learning model serving architecture with a bit more clarity. 

# Model Serving

Firstly, Rust ecosystem for Machine Learning is bad. Straight up, poor. Things like ONNX don't work if you try the ML extensions, dealing with types and file formats from "common" data file types don't quite work out of the box, and it feels like I'm trying to deploy something as though its 2015 rather than 2022. 

Regardless, the pattern I landed on is simply to recompile a model to Rust - not ideal, and would have lots of draw-backs in production. But lets just run with it for now. Perhaps this is a sensible way to slowly migrate workflows to Rust?

Libraries considered:
* onnx-rs
* tract

In Python:
* mcgen for generating Rust files

# Rust's Module System

Rust module system, although a bit confusing isn't _too_ bad. The general things you have to remember is things need to exist on the top level (i.e. in the `main.rs`) before it can be imported downstream and become exposed. So I ended up with `main.rs` file with a bunch of these at the top level:

```rs
mod handlers;  // handles logic flow
mod ml_models; // where machine learning evaluation resides
mod models;    // the data schemas such as API request/response objects
mod services;  // houses the business logic of the application
mod utils;     // auxiliary tools
```

Overall that's okay, and I would just treat it as part of the language. Then from there the folder structure needs a `mod.rs` and information on which functions are exposed publically via `pub ...`, and the usage across different modules would look like `use crate::services::...`. This kept code relatively clean and compact from my standpoint. 

I probably violated a whole bunch of Rust best practises along the way, but at least it feels "familiar" to me, and has a logical separation of where business logic may fall (in a DDD setting). 

# Things I'm not 100% across

Of course there are things that still confuse me, namely the difference between `&String` and `&str` - I can Google the solution, but ideally I would have the answer internalised (i.e. this is an issue in my "Code Fluency"). 

The concept of ownership did trip me up as well. Its obvious when you spell it out and think back, but its an aspect I need to continually remind myself. Specifically the patern was this:

```rs
let model_prediction = match model_name.as_ref() {
    "<model_name>" => <model_service>::predict(features)?,
    _ => "Model not found".to_string(),
};

// don't unwrap this! as you need to put it up for caching and return
cache_utils::save_in_cache(&cache_key, &model_prediction, ttl);
Ok(model_prediction)
```

Initially I tried unwrapping, and the returning and saving it into cache and not understanding why it wouldn't work. In my head this would operate because you would save the variable, and then copy it over, but in doesn't quite work that way in Rust (due to the nature of pointers and references). 

This doesn't work in Python, because it would be something like this:

```py
def predict(...):
    yield output

output = list(predict(...))  # evaluate the generator
cache_utils.save_in_cache(key, output)
return output # ...
```

Placing the generator in the save in cache would consume it and it wouldn't be available after. The ideas between the two languages is actually the same, its just about the execution and the nuance.

# What's next

I want to extend this to also consider Kafka, and setup services which split up the web server and the workers so they can be scaled up independently. This would also give me some exposure to placing sensible patterns end to end in a clean code manner.