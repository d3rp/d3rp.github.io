---
title: "Ranger Tmux Vim Split"
date: 2018-02-15T20:26:38+02:00
draft: true
---

map eh eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h 'vim " + fm.thisfile.basename + "'")
map ev eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v 'vim " + fm.thisfile.basename + "'")
map es eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -v -c $PWD")
map eS eval if 'TMUX' in os.environ: fm.execute_console("shell tmux splitw -h -c $PWD")
