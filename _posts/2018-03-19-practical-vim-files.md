---
layout: post
title: "Practical Vim: Files"
date: 2018-03-19 01:00:00
categories: en
tags: Vim
---

> Series notes on _Practical Vim_ by D. Neil:
> - [Practical Vim: Modes](/2018/03/17/practical-vim-modes/)
> - Practical Vim: Files
> - [Practical Vim: Getting Around Faster](/2018/03/20/practical-vim-getting-around-faster)
> - [Practical Vim: Registers](/2018/03/24/practical-vim-registers)

__Contents__

* content
{:toc}

## Chap 6 - Manage Multiple Files

### Tip 36: Track Open Files with the Buffer List

When we are editing a file, we are actually operating on an in-memory representation of a file, named _buffer_.

`:ls`: show the buffer list.

`:bnext`: switch to the next buffer in the buffer list, in which the `#` symbol represents the alternate file.

`<C-^>`: toggle between the current and alternate files.

`:buffer N`: jump to a buffer by number.

`:buffer {bufname}`: jump to a buffer by name. The `{bufname}` need only contain enough characters from the filepath to uniquely identity the buffer.

`:bufdo`: allows to execute an Ex command in all of the buffers listed by `:ls`.
In practice, `:argdo` may be more practical.

Deleting buffers can be in one of these forms:

```
:bdelete N1 N2 N3
:N,M bdelete
```

`:bdelete` can be written as `:bd`.

### Tip 37: Group Buffers into a Collection with the Argument List

`:args`: provides the argument list. 
The argument list represents the list of files that was passed as an argument when we ran the `vim` command.

`:args {arglist}`: the `{arglist}` can include filenames, wildcards, or even the output from a shell command. 
The technique works fine if we want to add some buffers.

__Specify Files by Glob__

| Glob            | Files Matching the Expansion                                                   |
|-----------------|--------------------------------------------------------------------------------|
| `:args *.*`     | `index.html`<br>`app.js`                                                       |
| `:args **/*.js` | `app.js` <br> `lib/framework.js` <br> `app/controllers/Mailers.js` <br> ...etc |
| `:args **/*.*`  | `app.js` <br> `index.js`<br> `lib/framework.js` <br> `lib/theme.css`<br>`app/controllers/Mailers.js` <br> ...etc |

__Specify Files by Backtick Expansion__

`` :args `cat .chapters` ``: execute the text inside the backtick characters in the shell, using the output from the `cat` command as the argument for the `:args` command.
Here, we get the contents of the hidden file `.chapters` and pass them to `:args`.

__Use the Argument List__

Argument list is simpler to manage than Buffer list.

- `:args {arglist}` can clear and repopulate the list with a single command
- `:next` `:prev` can traverse the files in the list
- `:argdo` can execute the same command on each buffer in the arg list

### Tip 38: Manage Hidden Buffers

| Command    | Effect                                                                         |
|------------|--------------------------------------------------------------------------------|
| `:w[rite]` | write the contents of the buffer to disk                                       |
| `:e[dit]!` | read the file from disk back into the buffer (that is, revert unsaved changes) |
| `:qa[ll]!` | close all windows, discarding changes without warning                          |
| `:wa[ll]`  | write all modified buffers to disk                                             |

Vim does not allow going to other items in Argument list until we save the changes in the current buffer. 
If `hidden` setting is enabled, then we can use the `:next` `:bnext` `:cnext` commands without a trailing bang.
If the active buffer is modified, Vim will automatically hide it when we navigate away from it.
The `hidden` setting makes it possible to use `:argdo` and `:bufdo` to change a collection of buffers with a single command.

### Tip 39: Divide Workspace into Split Windows

| Command            | Effect                                                                              |
|--------------------|-------------------------------------------------------------------------------------|
| `<C-w>s`           | split the current window horizontally, reusing the current buffer in the new window |
| `<C-w>v`           | split the current window vertically, reusing the current buffer in the new window   |
| `:sp[lit] {file}`  | split the current window horizontally, loading `{file}` into the new window         |
| `:vsp[lit] {file}` | split the current window vertically, loading `{file}` into the new window           |


| Command  | Effect                        |
|----------|-------------------------------|
| `<C-w>w` | cycle between open windows    |
| `<C-w>h` | focus the window to the left  |
| `<C-w>j` | focus the window below        |
| `<C-w>k` | focus the window above        |
| `<C-w>l` | focus the window to the right |

`<C-w><C-w>` does the same thing as `<C-w>w`; so as the other four in the table above.

| Ex Command | Normal Command | Effect                                          |
|------------|----------------|-------------------------------------------------|
| `:cl`      | `<C-w>c`       | close the active window                         |
| `:on[ly]`  | `<C-w>o`       | keep only the active window, closing all others |


| Keystrokes   | Buffer Contents                          |
|------------- |------------------------------------------|
| `<C-w>=`     | equalize width and height of all windows |
| `<C-w>_`     | maximize height of the active window     |
| `<C-w>|`     | maximize width of the active window      |
| `[N]<C-w>_`  | set active window height to `[N]` rows   |
| `[N]<C-w>|`  | set active window width to `[N]` columns |

### Tip 40: Organize Window Layouts with Tab Pages

A _tab page_ is a container that can hold a collection of windows.
Tab pages are available whether in GVim or inside a terminal. GVim draws a tab bar as part of the GUI, while in-terminal Vim draws a tab bar as a textual user interface (TUI).
Tab pages, apart from the differences in appearance, are functionally identical in GUI and TUI.

`:lcd {path}` set the working directory locally for the current window (not for the current tab page).
For a tab page containing multiple split windows, `:windo lcd {path}` can set the local directory for all the split windows.

| Command                 | Effect                                            |
|-------------------------|---------------------------------------------------|
| `:tabe[dit] {filename}` | open `{filename}` in a new tab                    |
| `<C-w>T`                | move the current window into its own tab          |
| `:tabc[lose]`           | close the current tab page and all of its windows |
| `:tabo[nly]`            | keep the active tab page, closing all others      |

| Ex Command       | Normal Command | Effect                          |
|------------------|----------------|---------------------------------|
| `:tabn[ext] {N}` | `{N}gt`        | switch to tab page number `{N}` |
| `:tabn[ext]`     | `gt`           | switch to next tab page         |
| `:tabp[revious]` | `gT`           | switch to previous tab page     |

`:tabmove [N]` can rearrange tab pages. When `[N]` is 0, the current tab page is moved to the beginning; when `[N]` is omitted, it is moved to the end.

## Chap 7 - Open Files and Save Them to Disk

### Tip 41: Open a File by `:edit`

`:pwd`: print working directory.

`:edit {filepath}`: open a file by its absolute or relative filepath.

`:edit %<Tab>`: the `%` symbol is shorthand for the filepath of the active buffer.
Pressing `<Tab>` expands the filepath, revealing the absolute path of the active buffer.

`:edit %:h<Tab>`: the `:h` modifier removes the filename while preserving the rest of the path.
So `%h<Tab>` will give the full path of the current file's directory.

We can creating a mapping for the `%:h` expansion like

```
cnoremap <expr> %% getcmdtype() == ':' ? expand('%:h').'/' : '%%'
```

Now when we type `%%` on Vim's `:` command-line prompt, it automatically expands to the directory path of the active buffer, just like `%:h<Tab>`.

### Tip 42: Open a File by `:find`

The `:find` command allows to open a file by its name without having to provide a fully qualified path.

`:set path+=app/**`: set a set of directories inside of which Vim will search when the `:find` command is invoked.
The `**` wildcard matches all subdirectories beneath the `app/` directory.

For the `path` setting, the treatment of `*` and `**` is slightly different (see `:h file-searching`) from _Populating the Argument List_.
The wildcards are handled by Vim rather than by the shell.

### Tip 43: Explore the File System with netrw

netrw is a plugin, but comes as standard with the Vim distribution.
The minimum requirement for `.vimrc` file to load plugins:

```
set nocompatible
filetype plugin on
```

`$ vim .`: show the contents of current directory in a regular Vim buffer.
Pressing `-` can open the parent directory.

| Ex Command | Shorthand | Effect                                                    |
|------------|-----------|-----------------------------------------------------------|
| `:edit .`  | `:e.`     | open file explorer for current working directory          |
| `:Explore` | `:E`      | open file explorer for the directory of the active buffer |

In addition to `:Explore`, `:Sexplore` and `:Vexplore` open the file explorer in a horizontal or vertical split window.

netrw can also create new files (`:h netrw-%`) or directories (`:h netrw-d`), rename existing ones (`:h netrw-rename`), or delete them (`:h netrw-del`).

netrw can even read and write files across a network, using protocols including `scp` `ftp` `curl` and `wget`. Loop up `:h netrw-ref`.

### Tip 44: Save Files to Nonexistent Directories

Create a new file in a directory that does not exist:

```
:edit madeup/dir/doesnotexist.yet
```

Vim creates a buffer for it, but it cannot be saved directly with `:w`.
In this case:

```
:!mkdir -p %:h
:write
```

The `-p` flag tells the `mkdir` to create intermediate directories.

### Tip 45: Save Files as the Super User

Edit hosts as a common user:

```
$ vim /etc/hosts
```

Save the file without leaving Vim:

```
:w !sudo tee % > /dev/null
```

Although Vim is still running as a common user, by using `:write !{cmd}` we can run `{cmd}` in the external shell as the superuser.
`%` in command-line mode represent the path of the current buffer. So the final part is actually `sudo tee /etc/hosts/ > /dev/null`.
This command receives the contents of the buffer as standard input, using it to overwrite the contents of the `/etc/hosts` file.

> The `> /dev/null` tail exists here because `tee`  writes to stdout AND a file, and we want to silence stdout.

Additionally, Vim detectes that the file has been changed outside, so Vim will prompts us to choose whether to keep the version in the buffer or load the version on disk. 
In this case, the file and the buffer happen to have the same contents.
