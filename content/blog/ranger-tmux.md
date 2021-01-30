---
title: "TMUX Heart Ranger"
date: 2018-02-15T20:26:38+02:00
author: "Joni Turunen"
---

[Ranger](https://github.com/ranger/ranger) is a console based file
explorer that uses vim bindings. This makes it a very intuitive choice
for any vimmer when they're looking for a file explorer. I use also use
tmux and was wondering, if instead of replacing the whole ranger view,
I could instead open up any of the files I want to edit in vim from
ranger, in a new tmux pane.

Here's a quick snippet to configure splitting a tmux window into panes from
within ranger so that when you create a new pane, it opens up either a new terminal or a vim
session.

Add to the ``~/.config/ranger/rc.conf``:

{{< highlight ranger >}}
map eh eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h 'vim " + fm.thisfile.basename + "'")
map ev eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v 'vim " + fm.thisfile.basename + "'")
map es eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v -c $PWD")
map eS eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h -c $PWD")
{{< / highlight >}}

Then while navigating onto a file in ranger (having of course started tmux),
pressing ``e`` opens a menu to select between ``h, v, s, S``. These will split
the view into panes (h)orizontally or (v)ertically and open the file in vim in
that pane. The latter options also split the pane either horizontally or
vertically, but the 's' stands for (s)shell. More specifically, 's' opens up a terminal in the
current directory splitting the pane horizontally and the capital 'S' does the
same, but splits the pane vertically.
