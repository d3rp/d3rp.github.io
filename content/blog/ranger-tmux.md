---
title: "TMUX Heart Ranger"
date: 2018-02-15T20:26:38+02:00
author: "Joni Turunen"
---

[Ranger](https://github.com/ranger/ranger) is a console based file explorer that
uses vim bindings. This makes it a very intuitive choice for any vimmer when
they're looking for a file explorer. I also use tmux and was wondering, if
instead of replacing the whole ranger view upon editing a file, I could open
them up in a newly split tmux pane. Here's a quick snippet to configure
splitting a tmux window into panes from within ranger.

Add to the ``~/.config/ranger/rc.conf``:

{{< highlight ranger >}}
map eh eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h 'vim " + fm.thisfile.basename + "'")
map ev eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v 'vim " + fm.thisfile.basename + "'")
map es eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v -c $PWD")
map eS eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h -c $PWD")
{{< / highlight >}}

Now navigating onto a file in ranger (having of course started it inside tmux)
and pressing ``e`` opens a menu to select between ``h, v, s, S``. These
consecutive choices will split the view into panes 
- (h)orizontally opening the file in vim
- (v)ertically opening the file in vim
- (s)hell in the working directory splitting pane horizontally
- (S)hell in the working directory splitting pane vertically
