---
layout: post
category : web micro log
tags :
---

Recently I began playing around with OCR using tesseract at work. Getting it to work proved to be a pain, since there was no administration rights when installing the software.

A brief outline of the approach is highlighted here.

`pypdfocr` was the package which was used to convert pdf's (or even non-pdf files could be used here). Based on the documentation, the external requirements that were used were:

*  Portable version of Tesseract ([download the "tesseract-XXX-win32-portable.zip"](https://sourceforge.net/projects/tesseract-ocr-alt/files/))
*  Ghostscript Portable, which is available from [PortableApps.com](http://portableapps.com/apps/utilities/ghostscript_portable)
*  ImageMagick portable, which is available in the [binary download section](http://www.imagemagick.org/script/binary-releases.php), look for "ImageMagick-XXX-portable-XXX.zip"
*  Poppler for Windows, which I used the unofficial [binaries located here](http://blog.alivate.com.au/poppler-windows/)

As a side note I really should learn how to build these unofficial binaries myself so I can learn how to redistribute it. This includes making my own suite of portable apps for personal use. This will probably be a project I work on myself in the future.
