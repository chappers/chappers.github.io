---
layout: post
category : web micro log
tags : [openshift, webpy, postgresql, webpy]
tagline: 
---




I've been giving Openshift a shot for a few days now. 

What I've got running:  

*	webpy  
*	PostgreSQL  

Annoyances I've had:  

*	Getting multiline text to run with proper escaping.  

This really is the only gripe I've had with Postgresql (not with Openshift). But I think enough is enough for now, I'll revisit you later!

Lessons learnt:  

*	Use `insert` in mingw32 to paste  
*	Escaping strings is a POA  
*	Using webpy on a framework better suited to Django or Rails means lots of experimentation (i.e. lack of experience on my part)  
*	pgAdmin for PostgreSQL is not a good client  
*	Port forward? (not good with these things) may be required to connect using external clients: `ssh -o "ExitOnForwardFailure=yes" UID -L LOCAL_PORT:ADDRESS:PORT`  
