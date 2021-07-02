---
layout: post
category : 
tags : 
tagline: 
title: Shortcodes in Hugo - Thinking about data lookup
---

Its now been over 7 years since starting out this blog, and I've been thinking about a refresh! In this process I've looked at Hugo, another static site generator. 

# Shortcode

Shortcodes are similar to Jinja templates which can inject and access variables. In our setup, if we want to access key-value data pairs so that we can easily manage links and other content en masse.

In our setup, we'll add some data to the `data/test.yaml` file. Let's say that it looks like:

```
key:
  name: "my value"
```

Then to access the variable in our shortcode we could use:


<!-- {% raw %} -->
```
{{- $scratch := newScratch }}
{{- $targetkey := .Get "key" }}
{{- range $key, $value := $.Site.Data.test }}
    {{- if eq $key $targetkey -}}
        {{- $scratch.Set "name" $value.name }}
    {{- end -}}
{{- end }}
```
<!-- {% endraw %} -->

**How would you use this?**

This can be used in a variety of ways, such as managing a central location of image references so that it can only be changed in one location.