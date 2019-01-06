---
title: "File Exploration in Vim"
date: 2018-02-06T22:45:48+02:00
author: "Joni Turunen"
tags: ["vim", "netrw", "nerdtree", "file explorer"]
series: ["On a Vim"]
---

There's a heap of ways and a plethora of plugins for vim to help navigate file systems. Here's a few examples of ways I've found useful divided in two sections: accessing local vs remote files.

## Accessing Local Files

**Plugins and External Tools**

I use [ranger](https://ranger.github.io/) so it felt really convenient for a long while to [split a pane in tmux from within ranger]({{< ref "ranger-tmux.md" >}}) when I wanted to edit a file and then hit [ctrl-p](https://kien.github.io/ctrlp.vim/) when I wanted to navigate within that file's directory - which is basically what I often end up doing after editing the file in question. Ctrl-p opens a interactive view that updates upon every key insert and pattern matches the written characters to the files recursively in the directory you're at. It also allows quick commands to open those files in different tabs etc. It is still in my toolbox, but I've used it less and less as the time goes on. The main reason for this is that I found netrw.

**Netrw is in Vim by Default**

George Ornbo showed a good alternative work flow to ctrl-p (or in fact [nerdtree](https://github.com/scrooloose/nerdtree)) with using [netrw instead of nerdtree](https://shapeshed.com/vim-netrw/#netrw-the-unloved-directory-browser). This approach is intriguing as **netrw is a stock plugin shipped with the default installation of vim** and it seems to work pretty well for this kind of file navigation. However the most useful feature related to this is explained below in the "Accessing Remote Files" section. The short summary of Ornbo's post is that netrw uses 3 commands: ``:explore``, ``:Sexplore`` and ``:Vexplore`` which open netrw in the current buffer or by splitting the window horizontally or vertically. The post goes on for a few nifty configuration tweaks to make the netrw a little more usable - of which the most useful to me has been to configure netrw to show a tree view by default (``let g:netrw_liststyle = 3``). 

**I Whip My Files Back and Forth..**

Then there's vim's basic file navigational features.. Pressing ``gf`` when the cursor is over a file name - or a relative path - opens a file in the current working directory, if such file can be found. This can be tested easily by "cd"ing into a directory with files, opening vim, running ``:read !ls`` - which pastes the output of ``ls`` in the current buffer - and placing the cursor on top of any of the files and pressing ``gf``.

Vim holds a list of previously opened [buffers](https://romainl.github.io/the-patient-vimmer/2.html#_buffers). Now if several files have been opened during the vim session, they can be shuffled through back and forth by ``Ctrl-i`` and ``Ctrl-o`` (in the normal mode). ``:b <Tab>`` (notice the space) opens buffers which are shown by name (enter opens the shown buffer). 

Additionally, instead of going through buffers one-by-one, they can be enlisted with ``:ls`` which enumerates the list of buffers. To select any of these listed ones: ``:b <number><enter>``. These could be mapped together to show a list of buffers (i.e. like open files) and waiting for input for selecting the file to be edited by number:
{{< highlight vim >}}
:map <Leader>B :ls<CR>:buffer 
{{< / highlight >}}

**Quick Fixes**

If you haven't tried **vimgrep** before, here's a quick tour. Vimgrep greps matching a pattern to the files like grep, but within vim. The biggest incentive to use it over grep is that it creates a list of files that can be opened in another window and shuffling through the files is a breeze in it. Try it out:

``:vimgrep /word/gj **/*.*``

After running that command, open the "quick fix window" with:

``:cwindow`` or just ``cwin``

The rest should be quite intuitive. Navigating through the list pressing enter on the shown line opens that file on that line in a new buffer. In the vimgrep command above, the "word" refers to the pattern and ``**/*.*`` to matching it to any files recursively from subfolders of the current working directory.

Here are some more ways to handle files similarily to vimgrep from vim: [Compile, build and error check workflow](https://gist.github.com/ajh17/a8f5f194079818b99199)

## Accessing Remote Files

I mentioned netrw in the previous section. The biggest factor in using netrw for me is that it works similarily for remote files as it does for local files. The added value? If you've weaved yourself an elaborate web of plugins and configured your keybindings reducing the overhead(ache) of the vim defaults and want to have all that power at hand when accessing remote files - now you can. This way you can edit the files as if they were on your machine by typing something like:

``vim scp://remote_host/``

So how can we make this happen? For convenience, lets make some changes to the host resolving configurations. You might have several ssh keys for different services/servers.

{{< highlight sh >}}
# file: ~/.ssh/config
Host my_remote_server       # a symbolic name of your choice
  Hostname server_IP_or_DNS # the server URI
  User username             # the username for the server
  IdentityFile ~/.ssh/a_suitable_ssh_key_if_any # key for authentication
 
{{< / highlight >}}

So for example:
{{< highlight sh >}}
Host HAL
  Hostname 10.10.1.2
  User dave
  IdentityFile ~/.ssh/id_rsa
{{< / highlight >}}

After which netrw handles navigation when we use vim over ssh thusly:
{{< highlight bash >}}
vim scp://HAL/filename
{{< / highlight >}}
( source: [stackexchange](https://unix.stackexchange.com/questions/315844/editing-and-compiling-files-on-a-remote-server-with-vim) )

What should open before your eyes is the file imaginatively named 'filename' located on the remote host, in your (remote) user's **home directory**.

As netrw works like a filebrowser in a terminal, it's possible to leverage it to show the root of your home directory on the remote host as a file tree with just:
{{< highlight bash >}}
vim scp://my_remote_server/
{{< / highlight >}}

Here lies rendered a more complete version of this command, which in this case would use an absolute path - hence the double slashes before the path:
{{< highlight bash >}}
vim scp://username@server_IP_or_DNS[:port]//absolute_path/to/file.cpp
{{< / highlight >}}

All is well in the kingdom, but then - on some unsuspecting moment - vim comes back at you with:
``(netrw) not a netrw-style url; netrw uses protocol://[user@hostname[:port]/[path]``

"Oh you, vim..", you chuckle, as the harmless looking oneliner fails to scare you. Waiting for the netrw's file view - which never opens - the oneliner begs for a closer look and soon enough provokes a double-take when you get to read the part about the protocol. Metaphorical gears be crackling while mashing together abstractions of ssh configurations, port numbers, vimrc lines and all other little details that might have taken the wrong typo. Here's a tip from a frustrated blogger, that spent far too long on this: check that the host name ends in a slash '/'.

