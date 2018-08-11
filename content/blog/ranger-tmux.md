---
title: "Split Tmux Window from Ranger"
date: 2018-02-15T20:26:38+02:00
author: "Joni Turunen"
---

Here's a quick configuration snippet to split a tmux window into panes from within ranger so that the new pane opens up either a new terminal or a vim session.

Add to the ``~/.config/ranger/rc.conf``:

{{< highlight ranger >}}
map eh eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h 'vim " + fm.thisfile.basename + "'")
map ev eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v 'vim " + fm.thisfile.basename + "'")
map es eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v -c $PWD")
map eS eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h -c $PWD")
{{< / highlight >}}

Then while navigating onto a file in ranger (inside tmux), pressing ``e`` opens a menu for selecting between ``h, v, s, S``. In order those will split horizontally or vertically for vim or splitting the window into a terminal in the current directory either vertically or horizontally (in that order).
