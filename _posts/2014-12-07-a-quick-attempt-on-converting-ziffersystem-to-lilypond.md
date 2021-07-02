---
layout: post
category : web micro log
tags : 
---

If you are to look at the collection of guitar music which my dad owns, you will find that a fair amount of it is handwritten using the ziffersystem. This is due to his very strong sense of relative pitch. The the natural question is how might we convert this to a more traditional system? Indeed you can do it using [Jianpu-ly](http://people.ds.cam.ac.uk/ssb22/mwrhome/jianpu-ly.html) (at least judging by the website, I haven't actually tried it myself). But I thought I could quickly implement something that "works" using Pyparsing in Python. So here it is!

Below is my thoughts on implementing such a system, but note that it doesn't support everything. A list of things which are missing include:

*  Key and time signatures
*  Tuplets
*  Repeats
*  Any kind of slur or tie markings
*  Chords
*  Lyrics
*  Dynamics

The only focuses are on pitch and duration, with the simple assumption that everything is in C major.

# Parsing

Using [Pyparsing](http://pyparsing.wikispaces.com/) is really easy once you get a hang of how to use it. 

## Pitch

```python
pitch = (Literal('0') | Literal('1') | Literal('2') | Literal('3') | 
         Literal('4') | Literal('5') | Literal('6') | Literal('7')).setResultsName("pitch")
```

What is happening here?

We are asking Pyparsing to look for a single character from 0-8, and once it has been parsed I want to set the result names to be called "pitch". We can the same thing for accidentals:

```python
accidentals = (Literal('#') | Literal('b') | Literal('@')).setResultsName("accidental")
```

## Duration

Dealing with duration is slightly different, because we want to have one or more dashes or one or more underscores (underscores would be our underlining).

```python
minim = OneOrMore('-') 
quaver = OneOrMore('_') 
dotted = Literal('.')
```

Pyparsing allows us to group things and use the `Optional` command to indicate the information might be optional!

```python
duration = (Optional(minim | quaver) + Optional(dotted)).setResultsName("duration")
```

We can put all this together to build a single note, and then one or more of these notes to build music:

```python
Note = Combine(Optional(accidentals) + pitch + Optional(duration))
Music = OneOrMore(Group(Note))
```

We want to group each note so that we can iterate over it.

# Converting to LilyPond

To convert to lilypond music, we have to create some mappings:

```python
note_mapping = """0 r
1 c
2 d
3 e
4 f
5 g
6 a
7 b""".split('\n')
note_mapping = dict([tuple(x.split()) for x in note_mapping])

duration_mapping = {'':'4',
                    '---': '1',
                    '-': '2',
                    '_': '8',
                    '__' : '16'}

accidental_mapping = {'#' : 'sharp',
                      '@' : 'flat',
                      'b' : 'flat'}
```

These all relate the lilypond specific notation, for example, the mapping from `0:r` refers to how rests are labelled `r` in lilypond. 

Next we must think how we can convert a single note, and then use that as the basis to iterate over all our music:

```python
def convert_pitch(note):
    return note_mapping[note['pitch']]

def convert_duration(note):
    """converts note to the right thing"""
    dur_note = ''.join(note['duration'].asList())
    if dur_note == '':
        return duration_mapping[dur_note]
    elif dur_note[-1] == '.':
        return duration_mapping[dur_note[:-1]] + '.'
    else:
        return duration_mapping[dur_note]
        
def convert_accidental(note):
    try:
        return accidental_mapping[note['accidental']]
    except:
        return ''

def convert_note(note):
    # order is pitch, accidental, duration
    return convert_pitch(note) + \
           convert_accidental(note) + \
           convert_duration(note)
```

There is nothing really tricky here, and it would be intuitive if you follow the output of pyparsing. As a test see what is the output of the following:

```python
Note.parseString("1")
Note.parseString("1_.")
Note.parseString("#1_.")
Note.parseString("@1--")
```

To actually create the music, we can simply use list comprehension into a predefined lilypond template and we're done.

```python
def convert_music(music_string):
    music = """\\language "english"

\\relative c' {
%s
}""" % ' '.join([convert_note(x) for x in Music.parseString(music_string)])
    return music
```

Below are some examples of the output:

**Mary had a little lamb**

```python
print convert_music("3 2 1 2 3 3 3- 2 2 2- 3 5 5- 3 2 1 2 3 3 3 3 2 2 3 2 1---")
```

```
\language "english"

\relative c' {
e4 d4 c4 d4 e4 e4 e2 d4 d4 d2 e4 g4 g2 e4 d4 c4 d4 e4 e4 e4 e4 d4 d4 e4 d4 c1
}
```

![Mary had a little lamb](/img/ziff/mary.png)


**Misc**

```python
print convert_music("3 2 1 @2 3 0 3- #2 2 2- 3 5 5- 3 2-. 1 2 3 3 3 3 2 2 3 2 1---")
```

```
\language "english"

\relative c' {
e4 d4 c4 dflat4 e4 r4 e2 dsharp4 d4 d2 e4 g4 g2 e4 d2. c4 d4 e4 e4 e4 e4 d4 d4 e4 d4 c1
}
```

![Misc](/img/ziff/misc.png)


