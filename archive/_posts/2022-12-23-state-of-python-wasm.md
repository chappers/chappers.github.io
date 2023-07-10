---
layout: post
category:
tags:
tagline:
---

I thought I'll put down some raw thoughts on Python and Wasm support. At the time of writing - this basically doesn't exist.

- CPython had some preliminary working examples, but it was purely experimental on Python 3.11 where nothing using C bindings worked
- RustPython has a WASI target "native", but I couldn't get it working, and required writing Rust code to inject custom modules (which is what we probably want)
- Pyodide supports wasm-emscripten and not wasi

One promising direction is to rely on GraalPython and use GraalVM to target wasm (though this is also currently experimental software.

I think WASI is the way of the future as a target, it can in theory remove dependencies on Docker and simplify deployments, as maybe you can use Fastly to deploy a whole application rather than having more customised setups. 

Regardless we're probably 5 years away from having this solved unless there is a concerted effort to push this forward.

