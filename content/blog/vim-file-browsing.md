---
title: "Netrw is the new nerdtree - Browse through buffers"
date: 2018-02-06T22:45:48+02:00
draft: false
---

After finding [ctrl-p](https://kien.github.io/ctrlp.vim/) I thought it solved my navigation issues with vim. Additionally, it's convenient to split a pane in tmux from within [ranger](https://ranger.github.io/) and then continue with ctrl-p. George Ornbo showed a better way though: [netrw instead of nerdtree](https://shapeshed.com/vim-netrw/#netrw-the-unloved-directory-browser)

It is better, because if you traverse towards the depths of the root of the filesystem, ctrl-p might hang for what seems like an eternity on my old Thinkpad (am I completing the stereotype?). One could use nerdtree similarily, but I felt it breaks the magic with one more plugin. Apparently netrw is a stock plugin shipping with the default vim and has plenty of tricks up its sleave.

You might have several ssh keys for different services/servers. Define the host in **~/.ssh/config**:
{{< highlight bash >}}
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

Furthermore a more complete version of the command is:
{{< highlight bash >}}
vim scp://username@server_IP_or_DNS[:port]//full_path/to/file.cpp
{{< / highlight >}}

When opening a file, vim contains its path in its own little path. If so - ``gf`` opens a file that has been written under the cursor. So if one navigates on top of 'main.py' and the directory has such a file, ``girlfriend`` - sorry - ``gf`` opens that file.

Now if several files have been opened thusly, they can be shuffled through back and forth by Ctrl-i and Ctrl-b. Also the command mode accepts ``:b <Tab>`` (notice the space) in which case the open buffers will be shown by name (and enter opens the shown buffer).

Instead of going through buffers one-by-one, they can be enlisted with ``:ls`` which enumerates the list of buffers. To select any of these listed ones: ``:b <number><enter>``.

Also this: [some github](https://gist.github.com/ajh17/a8f5f194079818b99199)
