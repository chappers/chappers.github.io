---
layout: post
category:
tags:
tagline:
---

There are two projects currently on my mind:

* streamlit, but can be hosted easily on Flask/Django/FastAPI using htmx
* PNGTubing but more "open"

**Streamlit**

Streamlit is overly complicated to host for what it is. My hypothesis is there is a way to create something using the fantastic `htmx` library to achieve a simplier version that can be co-hosted with other API frameworks. I'm not the first to consider this, and some prototypes have already been built though.

I still think there is something here that can be untangled better, e.g. how do we handle Authentication? Authorization?

I'm definitely interested in this space to lower the bar for creating web applications

Edit: December 2022 - Pynecone is an upcoming project that might fit the bill! Though I may still create my own.

**PNG Tubing**

When I look at all the face-rigging etc, I think that its way too complicated for what most people _actually_ want, and that is a simple image that moves when you talk. Discord has a nice setup, but I'm surprised by the lack of open source tools that do this. My wish is to maybe create a cross-platform rust-based application that can accomplish what tools like "Gazo Tuber" and "veadotube" achieve. 

Edit: December 2022 - Cathode tube is a FOSS project that fills in this gap! https://cathode.tube/