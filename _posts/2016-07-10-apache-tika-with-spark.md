---
layout: post
category: web micro log
tags:
---

Since there isn't really any tutorial on this (possibly because its too simplistic)
here is some starter code for working with automatic document conversion. Apache Tika will automatically
guess file formats based on the MIME type. This is well documented on the Tika site, and easy to use:

```java
val content = new BodyContentHandler
val parser = new AutoDetectParser
val metadata = new Metadata
val stream = new BufferedInputStream(new FileInputStream(path))

parser.parse(stream, content, metadata)
```

If we are after plain text from the body content, then we simply can do:

```java
content.toString
```

Which will output the converted document to plain text.

We can then also map the metadata with the content, as the meta data is simply
contained in the `metadata` variable.

In this case we would simply have to extract each part of the metadata.

```java
metadata.names map( x => x -> metadata.get(x) ) toMap
```

If we want to wrap it all in one immutatable map, we can sort of "cheat"

```java
metadata.add("content", content.toString)
metadata.names map( x => x -> metadata.get(x) ) toMap
```

Thus we have in ~14 lines of code have parsed any document type with metadata and the content as a map:

```java
import org.apache.tika.metadata.Metadata
import org.apache.tika.parser.AutoDetectParser
import org.apache.tika.sax.BodyContentHandler
import org.apache.tika.parser._
import java.io._

val content = new BodyContentHandler
val parser = new AutoDetectParser
val metadata = new Metadata
val stream = new BufferedInputStream(new FileInputStream(path))

parser.parse(stream, content, metadata)
metadata.add("content", content.toString)
val singleMapEle = metadata.names map( x => x -> metadata.get(x) ) toMap
```
