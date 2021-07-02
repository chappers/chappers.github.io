---
layout: post
category : 
tags : 
tagline: 
---

Sometimes you just don't want your PDFs to be searchable. Maybe you've added some annotations to strike-through or block out some sensitive data (but you can still select it!). 

This can be resolved through  some CLI applications:

```
convert: topdf

topng:
	pdftoppm -png $(PDF) document

topdf: topng
	rm -r -f tmp/
	mkdir tmp
	mv document-*.png tmp
	mogrify -format pdf tmp/*.png
	pdftk tmp/*.pdf cat output $(PDF)-new.pdf

clean:
	rm -r -f *.png
	rm -r -f tmp/
```

Where the usage is:

```
make clean
make PDF="mypdf.pdf" convert
```

In here the steps are roughly:

*  Convert `pdf`s pages to individual `png`: `pdftoppm -png $(PDF) document`
*  Convert the individual `png` to individual `pdf`: `mogrify -format pdf tmp/*.png`
*  Collate all `pdf` into one big pdf: `pdftk tmp/*.pdf cat output $(PDF)-new.pdf`

This would let us create a new pdf file with no textual data!