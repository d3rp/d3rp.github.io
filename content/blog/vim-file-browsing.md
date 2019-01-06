---
title: "File Exploration in Vim"
date: 2018-02-06T22:45:48+02:00
author: "Joni Turunen"
tags: ["vim", "netrw", "nerdtree", "file explorer"]
series: ["On a Vim"]
---

After finding the [ctrl-p](https://kien.github.io/ctrlp.vim/) plugin I thought it solved my file navigation issues with vim. It felt as convenient as could be to [split a pane in tmux from within ranger]({{< ref "ranger-tmux.md" >}})  and then continue with ctrl-p. George Ornbo showed a better way though: [netrw instead of nerdtree](https://shapeshed.com/vim-netrw/#netrw-the-unloved-directory-browser). ( See [ranger](https://ranger.github.io/) if you like textual terminal based interfaces
for file navigation with vim bindings.) It is better, because if you traverse towards the depths of the root of the filesystem, ctrl-p might hang for what seems like an eternity on my old Thinkpad (am I completing the stereotype with the terminals and Thinkpads?). One could use nerdtree similarily, but I felt it breaks the magic with one more plugin. **Netrw is a stock plugin shipping with the default vim** and has plenty of tricks up its sleave.

## Accessing Remote Files

You might have several ssh keys for different services/servers. Define the host in **~/.ssh/config**:
{{< highlight sh >}}
Host my_remote_server # a symbolic name of your choice
  Hostname server_IP_or_DNS # the real name
  User username # the username you use for login
  IdentityFile ~/.ssh/a_suitable_ssh_key_if_any # for authentication purpose…

{{< / highlight >}}

After which netrw handles navigation when we use vim over ssh:
{{< highlight bash >}}
vim scp://my_remote_server/file.py
{{< / highlight >}}
( source: [stackexchange](https://unix.stackexchange.com/questions/315844/editing-and-compiling-files-on-a-remote-server-with-vim) )

What should open before your eyes is the 'file.py' located on the remote host, in your (remote) user's **home directory**.

As netrw works like a filebrowser in a terminal, it's possible to leverage it to show the root of your home directory on the remote host by reducting the file.py:
{{< highlight bash >}}
vim scp://my_remote_server/
{{< / highlight >}}

All is well in the kingdom, but then - on some unsuspecting moment - vim comes back at you with:
``(netrw) not a netrw-style url; netrw uses protocol://[user@hostname[:port]/[path]``

"Oh you, vim..", you chuckle, as the harmless looking oneliner fails to scare you. Waiting for the netrw's file view - which never opens - the oneliner begs for a closer look and soon enough provokes a double-take when you get to read the part about the protocol. Metaphorical gears be crackling while mashing together abstractions of ssh configurations, port numbers, vimrc lines and all other little details that might have taken the wrong typo. Here's a tip from a frustrated blogger, that spent far too long on this: The first step is to check what is the last character of the given argument; If vim complains about the formatting in this manner, check that the host name ends in a slash '/'. See example above.

Here lies rendered a more complete rendition of this command, which in this case would use an absolute path - hence the double slashes before the path:
{{< highlight bash >}}
vim scp://username@server_IP_or_DNS[:port]//absolute_path/to/file.cpp
{{< / highlight >}}

#### So what?

At this point you might ask, what is the added value? The added value: If you've weaved yourself an elaborate web of plugins and configured your keybindings reducing the overhead(ache) of the vim defaults and want to have all that power at hand when accessing remote files - now you can. This way you can edit the files as if they were on your machine. Of course sudoing some /etc/ files might introduce obstacles especially if you have no read access with the user login in. One could try still opening the
remote file and writing with sudo by entering ``:w !sudo %`` in vim after the edits.

## Accessing Local Files

When opening vim, it holds a reference to its current working directory. ``gf`` opens a file that has been written under the cursor and will work if the path of it is relative (or absolute) to the vim's working directory ``:pwd``. So if one navigates on top of 'main.py' and the directory has such a file, ``gf`` opens that file. This can be tested easily by "cd"ing into a directory with files, opening vim, running ``:read !ls``, placing the cursor on top of any of the files and pressing ``gf``.

Now if several files have been opened thusly, they can be shuffled through back and forth by ``Ctrl-i`` and ``Ctrl-o`` (in the normal mode). ``:b <Tab>`` (notice the space) opens buffers which are shown by name (enter opens the shown buffer). 

Additionally, instead of going through buffers one-by-one, they can be enlisted with ``:ls`` which enumerates the list of buffers. To select any of these listed ones: ``:b <number><enter>``. These could be mapped together to show a list of buffers (i.e. like open files) and waiting for input for selecting the file to be edited by number:
{{< highlight vim >}}
:map <Leader>B :ls<CR>:buffer 
{{< / highlight >}}


Here are some more ways to handle files from vim: [Compile, build and error check workflow](https://gist.github.com/ajh17/a8f5f194079818b99199)
