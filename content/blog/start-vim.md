---
title: "Vim in 2 Minutes"
date: 2018-03-26T20:19:59+03:00
draft: true
---

- Record desktop with screenkey on. Write a bunch of dada with "quotes" and "we're in insert mode" using the mnemonics mentioned here.

Think quick: what do these acronyms stand for?

* rtfm
* irl
* tl;dr:
* lol

Did you by any chance voice these out silently as full words instead of the initials the acronyms are constructed from? The idea behind the key shortcuts of vim is that every action is a mnemonic of a sequence of keystrokes instead of conjointly pressed keys:

* 5w  = "Go to 5th word"
* ci" = "Change everything inside the hyphens"
* dte = "Delete until end of the word"
* i   = "Insert"

The last one refers to 'insert' as in "switch to the insert mode". Vim starts in 'Normal' mode from which we can get to the 'insert' mode by typing 'i'.

![diagram ftw](/img/vim_diagram_1st_step.png)

In 'insert' mode we can write text and we can get back out of the 'insert' mode with 'esc' or 'ctrl-c'.

- a diagram of modes illustrating what keys transission modes
![diagram ftw](/img/diagram.png)


Vim has different modes of operation and the shortcut keys may or may not act the same within each of those modes. I am refering to the 'insert' mode for actually writing and the 'normal' mode for all the trickery with the mnemonics.  This mode switching lends to the infamous confusion of "How to exit vim!", because most of us we're used to get straight to the "start typing" mode when we launch a word editing program instead some limbo state waiting for the user to start deciding what mode they want to go into. So as one will, after starting vim, typically first wonder why can't they write anything and after getting to the 'insert' mode - usually by accident
 after typing some additional character beyond individual keystrokes - it suddenly gets quite confusing: there's no menus and one needs to exit the 'insert' mode before they can actually exit the vim.

![Orly-ExitingVim](https://cdn-images-1.medium.com/max/1200/1*AD1e170YTJaiUBypv1H9Ow.jpeg)

So how do you exit vim? Through the 'Command-line' mode. That is the mode that let's one save files and exit among other things. 'Command-line' mode is accessible from the 'normal' mode by typing ':'.

- Another diagram with cli mode added

Now would be a really good time to go to the 'Command-line' mode and type in 'vimtutor' and follow that. It is a built in tutorial that takes a beginner vimmer through these concepts in detail.

- diagram with 'vimtutor' -> great success

There's a million tutorials in using vim, but they usually start with the concepts and introduce a bunch of details before getting to the basic core of the program. This was my take on introducing vim in a way, that a beginner could get to the basics as soon as possible, and work their way deeper from there on their own.
