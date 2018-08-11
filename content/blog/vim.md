---
title: "I'm With Vim"
date: 2017-12-14T22:54:43+02:00
draft: true
tags: ["vim", "cli", "tui", "terminal programs"]
series: ["vim stuff"]
---

<!-- Convert to rst with pandoc and fix formatting (headers, lists) -->

I'm With Vim
-----------------

Imagine a person on stage gesticulating unpredictably but firmly with his index fingers fencing imaginary fairies - or exclamating his conclusions - either way. While delivering poignant punchlines with an occasional underlining rhetoric question, imagine him making somewhat charicaturized examples ending them with a mildly smug smile as he nails the fairies. Imagine this person as a key player in creating the user interface you are currently using. He was in the center of the melting pot that was to brew wysiwyg editors, personal
computers and the ethernet standard. Amidst fencing fairies on stage, Alan Kay claimed that the touchscreen is the worst interface ever. Paraphrasing: an interface designed for everyone is not especially good for anyone. I would further argue that the mouse is not that good of an interface either.

Computer science is quite an arduous endeavour - even onerous at times (I've been reading War and Peace - ooh). With insensitizing oneself to the upcoming traumas of tumbling down learning curves while figuring the ins and outs of architectures and fencing all kind of buzzing words and lurking hype coming after the engineer's soul when she is persistently set to build the best solutions out there - why would such a super hero lower their expectations and grab some hardware interface with binary modes of interaction and an additional
arrow thrown on the screen as if to mock the user: "You have +100 keys on the keyboard, but how about instead fling this arrowy thing around and try to hit these tiny pixels"? If the keyboard offers a plethora of mapping options from keys to actions - together or in sequence - why would one opt for a flying arrow and clickity-click action - oops - missed it.. wait.. almost.. click.. no.. click.. damn it.. click ... aaaand ... draaag.. oh pfff... sigh.. let's click again..

Need I say after my pretentious and clumsy introduction justifying and establishing the supremacy of a keyboard when compared to a mouse, that I think vim is quite brilliant. My usual pattern of thought has changed from "Wouldn't it be cool if you could.." to "I'll search for how to do X with vim". Complementarily, there's a lot of [terminal based programs](https://xaizek.github.io/2016-08-13/big-list-of-vim-like-software/), that follow the vim keymappings, so one can apply what they've learned with vim to use elaborate shortcut keys and functionality outside of vim. If one comes from the windows background, it's like having similar keyboard shortcuts across all your programs - imagine if you've not experienced programs, that don't have ctrl+c and ctrl+v mapped as copy and paste, but instead, you have to learn a different combination for every program - and for that matter - learn all keyboard shortcuts anew for every application you use.

Short list of my favourite programs that use vim bindings:

* ranger
* zathura
* vimium / vimperator plugin for firefox

One Does Not Simply Quit Vim
============================


With that said - what's the infamous, but surprisingly typical first experience with vim? It has its sensible key sequences of which quitting the program is the first predicament for a young grasshopper to work their way out of. Then the beaten one should be adviced, that - even though I just ended up writing about learning these keys anew - ironically ctrl+c does not copy anything, and ctrl+v pastes nothing in vim. It's 'y' and 'p' for 'yank' and 'paste' - and that all the keyboard shortcuts are based on similar word-for-a-function mnemonics. So basically - it's all different. After venturing past those obstacles one arrives at a valley of opportunities.. She might take the *vimtutor* and *quickref* routes to travel further down the rabbit hole that the extensive help documentation is. On top of the new world of keyboard shortcuts, this "manual" reads like no other manual (info or man pages). It has its learning curve, for sure.

While dabbling with nifty word, sentence and paragraph editing options, the fresh vimster thinks of other enhancements - maybe one finds "the best" dotfiles that define a heap of key mappings and the going gets easier. Also there's heaps of plugins to enhance and facilitate processes in vim. Soon the vim experience is filled with custom mappings, plugins, fonts, highlighting themes, autocompleters that write the code for you, file browsers and whatnot. There's dotfiles lying around the internets and vim can be fleshed out to resemble an IDE for multiple languages.

So What's the Caveat?
=======================

Take a knee - I'll tell you a story. At one point, I started learning Java and with it I used an IDE. I went through a bunch of them with a bunch of other languages: Netbeans, Eclipse, Intellij Idea, Code-Blocks, Clion, QtCreator, PyCharm... (Dramatic pause with a gleam of nostalgia shimmering at the corner of my eye as I glare at the horizon and the setting sun) What ultimately turned me to use vim heavily was the erratic operation of these IDEs on an old laptop - which had no trouble doing most of the things those
IDEs offered with a fraction of the memory and cpu. Also, I didn't work on any big projects with milions of lines of code and a small nations worth of team mates - so setting up a project for which ever IDE flavour I was using at the time for every little thing I wanted to solve or try out seemed like an overkill. Instead I could fire off vim and get it done in an instance. Also, I've used vim *countless times on fixing something on a server, that has nothing else than an option to ssh into it* as it comes as default with almost all installations of linux.  But alas - after hacking and tweaking for the ultimate vim experience, suddenly the trusty, quick-n-snappy editor starts bloating up from additional features until reaching a threshold in mass attracting some orbiting moons. I'm somewhat just over this turning point hopefully steering my vim story towards less cumbered configurations. This can be done by leveraging its own features where applicable and appropriate. Also choosing the best plugins for the job helps. Of these experiences I'll be writing in the following posts.

Where Next
==========

[patient vimmer](http://romainl.github.io/the-patient-vimmer/1.html)
    If I had read this fantastic piece of vimery when I started using vim, the aforementioned superfluous plugin addiction had been prevented, I assume. It explains the concepts behind the windows and buffers, which will at some point be very relevant to get a better feel of navigating between files and within files.


<!--
Phrases to practise:

* words
    * sporadic
    * stabilize .. flow
    * evocation
    * startlingly
* phrases
    * calls to mind
    * imagined story revolving around
    * there is a close parallel between the issues with .. and those with the ..
    * .. that troubled him not at all ..
    * .. punch a button ..
    * .. behaviors that we would construe as creative ..
    * .. invaded the place en masse ..
    * He figured he could handle this

Tools for examples and visualizations:

* screenkey
* recordmydesktop
* asciinema
-->
