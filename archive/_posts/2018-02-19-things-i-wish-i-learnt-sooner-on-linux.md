---
layout: post
category:
tags:
tagline:
---

Without a doubt, I wish I learnt a bit of `vim` and `screen` earlier on in my Linux journey. Although I now use both tools on a semi-regular basis, I would still be considered an amateur - knowing only the basic commands. However, the truth is you only need the basic commands to get a lot of work done!

Here is my hack guide to using `vim` and `screen` in order to save yourself a lot of pain for the simple things; without trying to know or do everything in the terminal.

## vim

Vim has two modes: insert, command.

To go into insert mode from command mode press `i`. Thats all we're going to cover in this guide.

To go to command mode, press `esc`. From here there are probably only a handful of commands you need to know. Below are them in order of importance:

- `:q!` - quit without saving or the "whoops, lets try that again"
- `:wq` - save and quit
- `dd` - delete the line
- `dw` - delete the whole current word
- `u` - undo the last command

That's it! Of course `vim` has lots of rich commands which you could take further, but for now thats enough to be dangerous.

## screen

Screen can be initialised by simply running `screen`. To enter a command, use `ctrl+a`, then:

- `c`: create new shell
- `?`: help
- `k`: kill session
- `n`: next screen
- `0-9`: go to screen 0-9
- `ctrl+a`: toggle with previous screen

Screen is much easier to use simply because the help documentation presents you with all the possible commands! The above will probably be more than enough for most sessions in the terminal.
