---
title: "Fixing Misencoded Characters with Vim"
date: 2018-02-08T09:49:18+02:00
draft: true
author: "Joni Turunen"
---

There was some mysterious \<92\>, \<93\>, \<94\> and \<96\> tags in a painfully pretentious and clumsily condescending essay / diary entry / blog post I had written in the past (it wasn't stylistically cohesive either, as you can deduct from all the slashes). The encoding was latin[dos] as reported by vim which is probably because I used to use Windows. There was so many red flags in this equation that something had to be done - so I chose to fix those characters, as if that was the most burning issue in this ordeal. Frankly though, I should've been given the file a few runs with [shred](https://linux.die.net/man/1/shred) and thus sense a hint of closure with my past.

What does one do when they need to figure out how to work with a command line or a curses based tool? Ask stackoverflow. Or rather, google for a readily available answer. I mean, why contribute to the infosharing by asking relevant questions or checking answers in the docs oneself? \*cough\* So in short, I found [the answer on stackoverflow](https://stackoverflow.com/a/2801132). However I found it after landing on atomic object's blog post about [character encoding tricks for vim](https://spin.atomicobject.com/2011/06/21/character-encoding-tricks-for-vim/) that explains how to change
encodings and check for character codes. It is as if I had learned this before, but something that I learned later pushed it away.. As if that was a thing.

I usually validate the sed commands (that's what I call replacing in vim, because it basically is that) that I'm about to use with using find. 

Finding <92> characters:
{{< highlight vim >}}
/\%x92
{{< / highlight >}}

Replacing those characters with ':
{{< highlight vim >}}
:%s/\%x92/'/g
{{< / highlight >}}

Changing the encoding of the file after the changes to utf-8 (when in doubt, use [utf-8](https://en.wikipedia.org/wiki/UTF-8)):
{{< highlight vim >}}
:set ++enc=UTF-8
{{< / highlight >}}

ps. you can use undo after setting the encoding this way.
