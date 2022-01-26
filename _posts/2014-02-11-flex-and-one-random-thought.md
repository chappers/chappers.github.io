---
layout: post
category: web micro log
tags: [c]
---

After working on parsing SAS code over the last few weeks, I thought I'll revisit on
the tools I used. I used `pyparsing`. Theres nothing wrong with using `pyparsing`
(in fact it was immensely helpful to get things done quickly) but I thought it
would be a good opportunity to examine `flex` and `bison`. I'm
more interested in looking at and improving on my C code.

The structure of the lexer and grammar file isn't completely new to me, since I had
been exposed to `ply` through [Udacity course](https://www.udacity.com/course/cs262),
but it still took more time that I would have liked to get things "up and running".

So what was I trying to achieve?

1.  accept a file to parse (through arguments)
2.  If no file arguments were parsed, parse a pre-determined string in the program already

So here is a solution after consulting [stackoverflow](http://stackoverflow.com/a/9920524/1992167) and [flex manual](http://flex.sourceforge.net/manual/Simple-Examples.html).

{% highlight c %}
int main( int argc, char \*_argv )
{
++argv, --argc; /_ skip over program name \*/
if ( argc > 0 ) {
yyin = fopen( argv[0], "r" );
}
else {
char input[250] = "this is my input string and some numbers 1 2 123";

        /*Copy string into new buffer and Switch buffers*/
        yy_scan_string (input);

        /*Analyze the string*/
        yylex();

        /*Delete the new buffer*/
        yy_delete_buffer(YY_CURRENT_BUFFER);
    }
    yylex();

}
{% endhighlight %}

I have a long way to go to be proficient in C.
