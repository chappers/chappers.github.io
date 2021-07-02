---
layout: post
category : web micro log
tags : 
---

What is interesting when you actually start using things that you build beyond simply just toys.

Recently I had a conversation with a colleague on why you shouldn't use [regex to parse xml data](http://blog.codinghorror.com/parsing-html-the-cthulhu-way/). The main points for why it should be permissible are mostly around practicality, and that our jobs might be ad-hoc. This begged the question; if we weren't going to use a library and wanted to parse XML files how would we do it in R in particular. 

This allowed me to revist Ramble, to see if it was (somewhat) up for the task.

It was surprising to me, how easy it was to accomplish the task of tokenizing (though making it useful in practise would be a different story). The idea behind building it for XML is quite simple:

Parse:

1. Start tags
2. Then consider single tags
3. Then parse the attributes and quote strings
4. Finally parse the end tags

In the code I had I did nothing to ensure that the XML file was valid; since I merely tokenized the data.

Here is the code below:


```r

#' XML parser example

xml = '<complexType name="SubjectType">
    <choice>
        <sequence>
            <choice>
                <element ref="saml:BaseID"/>
                <element ref="saml:NameID"/>
                <element ref="saml:EncryptedID"/>
            </choice>
            <element ref="saml:SubjectConfirmation" minOccurs="0" maxOccurs="unbounded"/>
        </sequence>
        <element ref="saml:SubjectConfirmation" maxOccurs="unbounded"/>
    </choice>
</complexType>'


xmlParser <- (many(startTag %alt% singleTag) %then%
          many(endTag %alt% singleTag))


startTag <- (
  symbol("<") %then%
    identifier() %then% 
    many(attributes) %then%
  symbol(">") %using% function(x) {
    els <- unlist(c(x))
    #return(unlist(c(x)))
    return(list(name=els[2], all=els))
  }
)

endTag <- (
  symbol("</") %then%
    identifier() %then% 
  symbol(">") %using% function(x) {
    return(unlist(c(x)))
  }
  )

singleTag <- (
  symbol("<") %then%
    identifier() %then% 
    many(attributes) %then%
    symbol("/>") %using% function(x) {
      els <- unlist(c(x))
      return(unlist(c(x)))
    }
)

attributes <- (
  identifier() %then% 
    symbol("=") %then%
    quoteString
  )

quoteString <- (
  symbol('"') %then%
    many(satisfy(function(x) {return(!!length(grep('[^"]+', x)))})) %then%
    symbol('"') %using% function(x) {
      return(paste0(unlist(c(x)), collapse=""))
    } 
  )

xmlParser(xml)

```

