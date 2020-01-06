---
layout: post
title: "Practical Vim: Getting Around Faster"
date: 2018-03-20 01:00:00
categories: en
tags: Vim
---

> Series notes on _Practical Vim_ by D. Neil:
> - [Practical Vim: Modes](/2018/03/17/practical-vim-modes/)
> - [Practical Vim: Files](/2018/03/19/practical-vim-files)
> - Practical Vim: Getting Around Faster
> - [Practical Vim: Registers](/2018/03/24/practical-vim-registers)

__Contents__

* content
{:toc}

## Chap 8 - Navigate Inside Files with Motions

### Tip 46: Keep Your Fingers on the Home Row

_home row_: left hand on `a` `s` `d` `f`; right hand on `h` `j` `k` `l`.

| Command | Move cursor      |
|---------|------------------|
| `h`     | one column left  |
| `l`     | one column right |
| `j`     | one line down    |
| `k`     | one line up      |

### Tip 47: Distinguish Between Real Lines and Display Lines

When the `wrap` setting is enabled (it's on by default), each line that exceeds the window width will display as wrapped.

| Command | Move cursor                                 |
|---------|---------------------------------------------|
| `gj`    | down one display line                       |
| `gk`    | up one display line                         |
| `0`     | to first character of real line             |
| `g0`    | to first character of display line          |
| `^`     | to first nonblank character of real line    |
| `g^`    | to first nonblank character of display line |
| `$`     | to end of real line                         |
| `g$`    | to end of display line                      |

### Tip 48: Move Word-Wise

| Command | Move cursor                          |
|---------|--------------------------------------|
| `w`     | forward to start of next word        |
| `b`     | backward to previous 'start of word' |
| `e`     | forward to next 'end of word'        |
| `ge`    | backward to end of previous word     |

`ea` acts like 'append at the end of the current word'; `gea` acts like 'append at the end of the previous word'.

For each of the word-wise motions, there is a WORD-wise equivalent, including `W` `B` `E` `gE`.
A WORD is defined as consisting of a sequence of nonblank characters separated with whitespaces.

### Tip 49: Find by Character

| Command   | Effect                                                              |
|-----------|---------------------------------------------------------------------|
| `f{char}` | forward to the next occurrence of `{char}`                          |
| `F{char}` | backward to the previous occurrence of `{char}`                     |
| `t{char}` | forward to the character before the next occurrence of `{char}`     |
| `T{char}` | backward to the character after the previous occurrence of `{char}` |
| `;`       | repeat the last character-search command                            |
| `,`       | reverse the last character-search command                           |

Character search can be used like a motion, and hence can be combined with `d{motion}` `c{motion}` to finish more complicated operations.

It is better to choose target characters with a low frequency of occurrences, e.g. `x` `z` are better than `e` `f`.

### Tip 50: Search to Navigate

`/` to search for characters in the buffer in a forward direction; `?` in a backward direction.
Note that it jumps to put the cursor right at the beginning of the occurrence.

`n` to jump to the next occurrence by repeating the previous search; `N` to jump in the inverse direction.

Search can help in Visual mode to guide text selection.

Search can be combined with `d{motion}`.

### Tip 51: Trace Selection with Precision Text Objects

| Text Object  | Selection                    |
|--------------|------------------------------|
| `a)` or `ab` | a pair of `(parentheses)`    |
| `i)` or `ib` | inside of `(parentheses)`    |
| `a}` or `aB` | a pair of `{braces}`         |
| `i}` or `iB` | inside of `{braces}`         |
| `a]`         | a pair of `[brackets]`       |
| `i]`         | inside of `[brackets]`       |
| `a>`         | a pair of `<angle brackets>` |
| `i>`         | inside of `<angle brackets>` |
| `a'`         | a pair of `'single quotes'`  |
| `i'`         | inside of `'single quotes'`  |
| `a"`         | a pair of `"double quotes"`  |
| `i"`         | inside of `"double quotes"`  |
| ``a` ``      | a pair of `` `backticks` ``  |
| ``i` ``      | inside of `` `backticks` ``  |
| `at`         | a pair of `<xml>tags</xml>`  |
| `it`         | inside of `<xml>tags</xml>`  |

### Tip 52: Delete Around, or Change Inside

| Keystrokes | Buffer Contents                  |
|------------|----------------------------------|
| `iw`       | current word                     |
| `aw`       | current word plus one space      |
| `iW`       | current WORD                     |
| `aW`       | current WORD plus one space      |
| `is`       | current sentence                 |
| `as`       | current sentence plus one space  |
| `ip`       | current paragraph                |
| `ap`       | current paragraph plus one space |

### Tip 53: Mark Your Place and Snap Back to It

`m{a-zA-Z}`: marks the current cursor location with the designated letter (see `:h m`).

`` `{mark} ``: jump to a mark.

__Automatic Marks:__

| Keystrokes      | Buffer Contents                                   |
|-----------------|---------------------------------------------------|
| <code>``</code> | position before the last jump within current file |
| `` `. ``        | location of last change                           |
| `` `^ ``        | location of last insertion                        |
| `` `[ ``        | start of last change or yank                      |
| `` `] ``        | end of last change or yank                        |
| `` `< ``        | start of last visual selection                    |
| `` `> ``        | end of last visual selection                      |


### Tip 54: Jump Between Matching Parentheses

`%` lets us jump between opening and closing sets of parentheses (see `:h %`).
It works with `()`, `{}`, and `[]`.

Vim ships with a plugin _matchit_, which enhances the `%` command.
When _matchit_ is enabled, `%` can jump between matching pairs of keywords, like tags in HTML, `class/end` `def/end` `if/end` in Ruby.
To enable _matchit_ on startup:

```
set nocompatible
filetype plugin on
runtime macros/matchit.vim
```

Another good plugin is _Surround.vim_. Visually select some characters, and `S"` would surround the selection with a pair of `"`.
`S)` `S}` work similarly.
Changing existing delimiters is also possible: `cs}]` would change `{London}` to `[London]`.
_Surround.vim_ should be mannually [installed](https://github.com/tpope/vim-surround).

## Chap 9 - Navigate Between Files with Jumps

### Tip 55: Traverse the Jump List

`:jump`: show the jump list

`<C-o>`: jump back

`<C-i>`: jump forward

| Command                                | Effect                                         |
|----------------------------------------|------------------------------------------------|
| `[count]G`                             | jump to line number                            |
| //pattern`<CR>`/?pattern`<CR>`/`n`/`N` | jump to next/previous occurrence of pattern    |
| `%`                                    | jump to matching parentheses                   |
| `(`/`)`                                | jump to start of previous/next sentence        |
| `{`/`}`                                | jump to start of previous/next paragraph       |
| `H`/`M`/`L`                            | jump to top/middle/bottom of screen            |
| `gf`                                   | jump to file name under the cursor             |
| `<C-]>`                                | jump to definition of keyword under the cursor |
| `'{mark}`/`` `{mark}``                 | jump to a mark                                 |

### Tip 56: Traverse the Change List

`:changes`: show the change list

`g;`: traverse backward in the change list

`g,`: traverse forward in the change list

`` `.``: mark to the position of the last change

`` `^``: mark to the position of the cursor the last time Insert mode was stopped

`gi`: use `` `^`` to restore the cursor position, and then switch to Insert mode


### Tip 57: Jump to the Filename Under the Cursor

`gf`: jump to the filename under the cursor

`:set suffixesadd+=.rb`: tell Vim to find filenames with `.rb` extension.
Note that common file-type extensions are automatically handled in most modern Vim distributions.

`:set path?`: inspect the value of `path`

### Tip 58: Snap Between Files Using Global Marks

`m{letter}`: create a mark at the current cursor position.
Lowercase letters work locally in a buffer;
Uppercases are global.

`:vimgrep /{pattern}/ {files}` can search and jump in files.
Set a global mark before diving with `:vimgrep`.

