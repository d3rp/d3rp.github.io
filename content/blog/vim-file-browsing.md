---
title: "File Exploration in Vim"
date: 2018-02-06T22:45:48+02:00
author: "Joni Turunen"
tags: ["vim", "netrw", "nerdtree", "file explorer"]
series: ["On a Vim"]
---

Vim is a great text editor. There's a plethora of third party plugins to extend
vim and help navigate file systems and also a heap of ways to use them. I'm not
sure, if it's common knowledge that vim has very nifty tricks up its sleeve for
to tackle exploring the directory structures without additional plugins and
opening up remote files using your beloved vim configuration. That is, if you
are inclined to heavily configure your vim with or without plugins, then all
that sweetness can be applied to editing files remotely instead of having to use
some remote machine's vanilla vim.

Here are some ways divided into two sections that I've found useful. I've
divided them according to their use case, which are "accessing local files" and
"accessing remote files".

## Accessing Local Files

**Plugins and External Tools**

I use [ranger](https://ranger.github.io/) and it felt really convenient for a
long while to [split a pane in tmux from within ranger]({{< ref "ranger-tmux.md"
>}}) when I wanted to edit a file and then hit
[ctrl-p](https://kien.github.io/ctrlp.vim/) when I wanted to navigate within
that file's directory inside vim - which is basically what I often end up doing
after editing the file in question.

Ctrl-p refers to the shortcut used as well as the name of the plugin enabling a
fuzzy search file exploration feature.  Hitting ctrl-p in vim with the plugin
installed opens an interactive view that updates upon every typed key and
pattern matches the written characters to the files recursively in the directory
you're at. It also allows quick commands to open those files in different tabs
etc. It is still in my toolbox, but I've used it less and less as the time goes
on.

The main reason for my abandoning of this plugin is that it sometimes gets
awkwardly stuck when there's too many files and folders to recurse into, and vim typically hangs
until the find process is finished without accepting any keyboard input.

Also having found netrw in vim made me steer towards using it for navigating
instead of ctrl-p.

**Netrw is in Vim by Default**

George Ornbo showed a good alternative work flow to ctrl-p (or in fact
[nerdtree](https://github.com/scrooloose/nerdtree)) with using [netrw instead of
nerdtree](https://shapeshed.com/vim-netrw/#netrw-the-unloved-directory-browser).
This approach is intriguing as **netrw is a stock plugin shipped with the
default installation of vim** and it seems to work pretty well for this kind of
file navigation. The short summary of Ornbo's post is that netrw uses 3
commands: ``:explore``, ``:Sexplore`` and ``:Vexplore`` and netrw view can be
toggled and configured to suit your taste. These commands open netrw in the
current buffer (explore) or by splitting the window horizontally (Sexplore) or
vertically (Vexplore).

His blog post goes on to introduce a few nifty configuration tweaks to make the
netrw a little more usable. Of these tweaks I found the most useful to be
configuring netrw to show a tree view by default (``let g:netrw_liststyle =
3``).  After mapping some keys, for example <Leader>x for explore etc. the work
flow becomes quite intuitive when you want to open another file from the working
directory.

However, I've explained what I feel is the most useful feature related to this
in this post in the section below (Accessing Remote Files). 

**I Whip My Files Back and Forth..**

There's also vim's basic file navigation features aside from netrw. Pressing
``gf`` when the cursor is over a file name - or a relative path - opens a file
in the current working directory, if such a file can be found. This can be tested
easily by "cd"ing into a directory with files, opening vim, running ``:read
!ls`` - which pastes the output of ``ls`` in the current buffer - and placing
the cursor on top of any of the files and pressing ``gf``. This would open the
file under the cursor in vim.

Every opened file is shown in a buffer. Vim holds a list of previously opened
[buffers](https://romainl.github.io/the-patient-vimmer/2.html#_buffers). Now if
several files have been opened during the same vim session, they can be shuffled
through back and forth by ``Ctrl-i`` and ``Ctrl-o`` (in the normal mode).
There's actually more to these two keyboard shortcuts - they take you back to
the exact position you jumped from regardless of if you moved inside the same
file or between files. It is quite equivalent to the back and forward arrows inside
most IDEs.

``:b <Tab>`` (notice the space) iterates over the buffers by pressing tab
completing their names (enter opens the buffer). 

Additionally, instead of going through buffers one-by-one, they can be enlisted
with ``:ls`` which enumerates the list of buffers. To select and open any of these enlisted ones: ``:b <number><enter>``. These could be mapped together to show a list of buffers (i.e. like open files) and waiting for input for selecting the file to be edited by number:
{{< highlight vim >}}
:map <Leader>B :ls<CR>:buffer 
{{< / highlight >}}

**Quick Fixes**

If you haven't tried **vimgrep** before, here's a quick tour. Vimgrep greps
matching a pattern to the files like running grep would, but within vim. The
biggest incentive to use this features over whipping open a terminal and using
grep instead is that it creates an index of files with the search results that can be opened in another
window. Then it is a breeze to go through through the search results and open
the file that is of interest.. Try it out:

``:vimgrep /word/gj **/*.*``

After running that command, open the "quick fix window" i.e. the index with:

``:cwindow`` or just ``cwin``

The rest should be quite intuitive. Navigating through the list pressing enter
on the shown line opens that file - placing the cursor on the line with the
search result - in a new buffer. In the vimgrep command above, the "word" refers
to the pattern and ``**/*.*`` to matching it to any files recursively from
subfolders of the current working directory.

Here are some more ways to handle files similarily to vimgrep from vim: [Compile, build and error check workflow](https://web.archive.org/web/20170531234737/https://gist.github.com/ajh17/a8f5f194079818b99199)

If you have a vim compiled with the make-tag, you can use vim's automatic
features to [open cwindow and examine
errors](https://vim.fandom.com/wiki/Automatically_open_the_quickfix_window_on_:make)
that resulted when compiling.

## Accessing Remote Files

I mentioned netrw in the previous section. The biggest ergonomics factor in
using netrw for me is that it works similarily for remote files as it does for
local files. The added value? If you've weaved yourself an elaborate web of vim
plugins and configured your keybindings reducing the overhead(ache) liberating
your text editing experience beyond the defaults thrown on you by a vanilla vim
installation - and you want to have all that power at hand when accessing remote
files - now you can.  This way you can edit the files as if they were on your
machine by typing something like:

``vim scp://remote_host/``

In effect, vim handles copying the file from the remote machine to your local
location and after saving the edits it will copy it back 
So how can we make this happen? For convenience, lets make some changes to the
ssh configurations dealing with resolving the host. I think this plays a major part
in making the work flow ergonomic for those remote edits.

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

After configuring the ssh hosts as shown above, netrw is able to handle
navigating directories and files, when we use vim over ssh thusly:
{{< highlight bash >}}
vim scp://HAL/filename
{{< / highlight >}}
( source: [stackexchange](https://unix.stackexchange.com/questions/315844/editing-and-compiling-files-on-a-remote-server-with-vim) )

What should open before your eyes is the file imaginatively named 'filename'
located on the remote host, in your (remote) user's **home directory** - in this
example's case the user is "dave".

As netrw works like a file browser in a terminal, it's possible to leverage it
to show the root of your home directory on the remote host as a file tree with
just:
{{< highlight bash >}}
vim scp://my_remote_server/
{{< / highlight >}}

If vim configuration included ``let g:netrw_liststyle = 3``, then the browsing
view will be one refering to a tree view.

Here lies rendered a more complete version of this command, which in this case
would use an absolute path - hence the double slashes before the path:
{{< highlight bash >}}
vim scp://username@server_IP_or_DNS[:port]//absolute_path/to/file.cpp
{{< / highlight >}}

As a last warning, note that the slashes in the scp-path have a confounding
syntax and might waste your time troubleshooting the issue. Here is some
tongue-in-cheek passage to explain why the slashes matter:

Let us conjure some imagery to address this potential side effect: All is well
in the kingdom, but then - on some unsuspecting moment - vim comes back at you
with: 

{{< highlight bash >}}
(netrw) not a netrw-style url; netrw uses protocol://[user@hostname[:port]/[path]
{{< / highlight >}}

"Oh you, vim..", you chuckle, as this harmless looking oneliner fails to waver
your intentions of connecting remotely.

Determined that the written path was perfectly adequate, you wait for the
netrw's file view to open - an alluring moment that persists beyond frustration
as the awaited tree view never presents itself - and so the error report begs for a closer
inspection.. And soon it provokes a double-take when the word "protocol" permeates
your dubious consciousness. After verifying the ssh configurations, port
numbers, vimrc lines and all other minute details that might have taken the
wrong typo, you'll realize what I've learned repeatedly upon making this same
mistake..

In short, check that the host name ends in a slash '/'.

