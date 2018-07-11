---
title: "Sudoku Haskell"
date: 2017-12-14T22:54:43+02:00
draft: true
---
While building an etl pipeline at work around the concept of coroutines... Reminded of the functional side of python..  remeniscing courses of scala and coming from a java background functional approaches called for me... But [scala|http://www.scala-lang.org/] doesn't enforce the functionality and after covering the basics I yearned for something more challenging... Remembered haskell

Family requiring its own time... I had only few hours to spend on this... Decided for some odd reason to take a notebook and go through the haskell 2010 syntax reference. 
https://www.haskell.org/onlinereport/haskell2010/haskellch2.html#x7-140002
Surprisingly it was some what beneficial, as usually when reading new languages, first issues tackle figuring out what's in the language domain and whats the application layer...

Next some general basics and installing some preliminary dev env. Tried the online interpreter but that didn't work on my phone. A helping outlook of haskelian things was this video explaining how haskell relates to cpu instructions with an C++ oriented audience: https://youtu.be/lC5UWG5N8oY

Checking simple stuff like
https://stackoverflow.com/questions/9142731/what-does-the-symbol-mean-in-haskell
Reminded me of the comic strip http://www.editgym.com/comics/00001.html (only for humor, has no validity whatsoever)
Ultimately finding this gem http://learnyouahaskell.com/types-and-typeclasses#typeclasses-101

Came by 'Maybe' and while taking an offtrack moment to look for old notes in my evernote, found category theory mentioned with Haskell and landed via its wikipedia page on https://en.wikipedia.org/wiki/Monad_%28functional_programming%29?wprov=sfla1

Instead of digging too deep into the breadth first approach and memorizing all basic concepts, I took on to write a sudoku solver. This is to evaluate what Ive read and mixing all pieces together into a story line.

I found examples of solvers so that was covered.
https://wiki.haskell.org/Sudoku
 I had done this before, but didnt remember much of it. Spent some time with pen and paper mapping out how this can be done in a stateless and functional way. This was to reduce the complexity of learning a new language and figuring out the mechanics simultaneously ( paradoxial problem of choices ).


