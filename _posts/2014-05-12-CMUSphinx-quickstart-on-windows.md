---
layout: post
category : code dump
tags : [CMUSphinx]
---

CMUSphinx is an open source toolkit for speech recognition. Here I have some notes
which helped me with getting started with CMUSphinx. This is not a comprehensive
tutorial, but rather just to give you the bare minimum to get it up and running.

1. On the [download page](http://cmusphinx.sourceforge.net/wiki/download/), download (at the time of writing, it was version `sphinxbase-0.8` and `pocketsphinx-0.8`): sphinxbase (windows binaries) and pocketsphinx (windows binaries).  
2.  Extract them in the same folder (say CMUSphinx), and rename them so that the folder
    only has two folders called "pocketsphinx" and "sphinxbase"

And that's it! Run the appropriate command and you can convert speech to text.

### Example

    pocketsphinx_batch.exe -hmm (hmm loc) -lm (lm loc) -dict (dic loc) -ctl (ctl loc) -cepext (file extention) -adcin true -hyp (output file)  

*   hmm : hidden markov model, e.g. `../../model/hmm/en_US/hub4wsj_sc_8k`
*   lm : language model, e.g. `../../model/lm/en/turtle.DMP`
*   dict : phonetic diction, e.g. `../../model/lm/en/turtle.dic`
*   ctl : control file, contains list of all the audio files which you wish to convert
*   cepext : the extension of the audio files you are converting
*   adcin : are the inputs audio files
*   hyp : hypothesis; the most likely sentence output

Through testing it out (with my non-American accent) it seems to have a faily poor conversion rate. I will
have to play with it more to determine how useful it is.

