---
layout: post
title: "Practical Vim: Modes"
date: 2018-03-17 01:00:00
categories: en
tags: Vim
---

> Series notes on _Practical Vim_ by D. Neil:
> - Practical Vim: Modes
> - [Practical Vim: Files](/2018/03/19/practical-vim-files)
> - [Practical Vim: Getting Around Faster](/2018/03/20/practical-vim-getting-around-faster)
> - [Practical Vim: Registers](/2018/03/24/practical-vim-registers)

__Contents__

* content
{:toc}


## Chap 1 - The Vim Way

### Tip 1: Repeat with Dot Command

`.`: Repeat last _change_

`>G`: indent lines until the end of file

### Tip 2: Append line and repeat

`A`: append to the end of current line

`C` = `c$`: delete until the end of line, into Insert mode

`s` = `cl`: delete the character at right, into Insert mode

### Tip 3: Forward scan and repeat

`f+`: forward scan line for character `+`

`;`: forward scan line for latest forward-scanned character

`;.`: forward scan line for latest forward-scanned character, repeat lately executed change

### Tip 4: Repeat and Reverse

|Intent                           | Act                   | Repeat | Reverse |
|---------------------------------|-----------------------|--------|---------|
|make a change                    | {edit}                | `.`    | `u`     |
|scan line for next char          | `f{char}`/`t{char}`   | `;`    | `,`     |
|scan line for previous char      | `F{char}`/`T{char}`   | `;`    | `,`     |
|scan document for next match     | /pattern`<CR>`        | `n`    | `N`     |
|scan document for previous match | ?pattern`<CR>`        | `n`    | `N`     |
|perform substitution             | :s/target/replacement | `&`    | `u`     |
|execute a sequence of changes    | `qx{changes}q`        | `@x`   | `u`     |


### Tip 5: Find and replace by hand

`*`: find the next string matched to the word below cursor

`n`: find the next matched string searched lately, which can follow the `/` or `*` command

`cw`: delete to the end of the word, into Insert mode.

`n.`: go to next matched string searched lately, repeat the lately executed change

### Tip 6: Revisit the Dot Formula

The Ideal: One Keystroke to Move, One Keystroke to Execute


## Chap 2 -  Normal Mode

### Tip 8: Chunk Your Undos

`i{insert some text}<Esc>` constitutes a change that can be undone by a `u` stroke.
Therefore, in Insert Mode, when openning a new line, try pressing `<Esc>o` instead of `<CR>` to keep the granularity in the change flow.

__Special case:__ If we use the `<Up>`, `<Down>`, `<Left>`, or `<Right>` cursor keys while in Insert mode, a new undo chunk is created.
It’s just as though we had switched back to Normal mode to move around with the `h`, `j`, `k`, or `l` commands, except that we don’t have to leave Insert mode.
This also has implications on the operation of the dot command.

### Tip 9: Compose Repeatable changes

`aw`: a word

`daw`: delete a whole word (this `daw` can be repeated as one step using `.`)

### Tip 10: Use Counts to Do Simple Arithmetic

The `<C-a>` and `<C-x>` commands perform addition and subtraction on numbers (at or after the cursor).
`18<C-a>` will add 18 to the number. When run without a count, they increment by one.

### Tip 11: Count vs Repeat

__Repeat over Count__: `dw.` may be better than `2dw` or `d2w` in: 
1. counting is tedious 
2. it can be undone by `u` with one word in one step; 
3. `u` and `.` commands have more granularity

__Count over Repeat__: 
1. if you want to change 3 words to another series of characters, `c3w` followed by those characters would be better than `dw..` and `i` and those characters
2. a clean and coherent undo history

### Tip 12: Combine and Conquer

__Operator + Motion = Action__:

The `d{motion}` commands can be like 
1. `dl` delete a character
2. `daw` delete a whole word
3. `dap` delete an entire paragraph

The same goes for `c{motion}`, `y{motion}`, etc. Find a complete list of operators by `:h operator`.
Some of them are:

| Trigger | Effect                                           |
|---------|--------------------------------------------------|
| `c`     | change                                           |
| `d`     | delete                                           |
| `y`     | yank into register (copy)                        |
| `g~`    | swap case                                        |
| `gu`    | make lowercase                                   |
| `gU`    | make uppercase                                   |
| `>`     | shift right                                      |
| `<`     | shift left                                       |
| `=`     | autoindent                                       |
| `!`     | filter {motion} lines though an external program |

One more rule: invoking an operator in duplicate would act upon the current line, like
1. `dd` deletes the current line
2. `>>` indents the current line
3. `gUgU` or `gUU` make the current line all uppercase

__Custom Operators__

Check by `:h :map-operator`

## Chap 3 - Insert Mode

### Tip 13: Corrections in Insert Mode

| Keystrokes | Effect                                |
|------------|---------------------------------------|
| `<C-h>`    | delete back one character (backspace) |
| `<C-w>`    | delete back one word                  |
| `<C-u>`    | delete back to start of line          |

These commands can also be used in Vim's command line as well as in the bash shell.

### Tip 14: Back to Normal Mode

| Keystrokes | Effect                |
|------------|-----------------------|
| `<Esc>`    | to Normal mode        |
| `<C-[>`    | to Normal mode        |
| `<C-o>`    | to Insert Normal mode |

Insert Normal mode is s special version of Normal mode.
It allows one command to execute, after which it will return to Insert mode immediately.

For example, the `zz` command redraws the screeen with the current line in the middle of the window.
Thus, in Insert mode, `<C-o>zz` helps to move our input flow into the middle of the window.

### Tip 15: Paste without Leaving Insert Mode

`<C-r>{register}` paste those in {register} in Insert mode.

`<C-r><C-p>{register}` is smarter, which inserts text literally and fixes any unintended indentation.
But it's a bit of handful. When pasting a register containing  multiple lines of text, consider switching to Normal mode.

### Tip 16: Do Calculations in Place

The _expression register_ allows us to perform calculations directly. It is addressed by the `=` symbol. 

In Insert mode, `<C-r>=`6*35`<CR>` will insert 210 directly in the text.


### Tip 17: Insert Unusual Characters by Character Code 

In Insert mode, `<C-v>{code}` can input the character whose address is {code}. 

`<C-v>u{00bf}` will insert the character whose unicode address is 00bf.
It should be a four-digit hexadecimal code.

`ga` outputs a message showing the address of the character under cursor.

| Keystrokes            | Effect                                                   |
|-----------------------|----------------------------------------------------------|
| `<C-v>{123}`          | insert character by decimal code                         |
| `<C-v>u{1234}`        | insert character by hexadecimal code                     |
| `<C-v>{nondigit}`     | insert nondigit literally                                |
| `<C-k>{char1}{char2}` | insert character represented by `{char1}{char2}` digraph |

### Tip 18: Insert Unusual Characters by Digraph

`<C-k>?I`: the ¿ character.

`<C-k>12`: the ½ character.

`<C-k>>>`: the » character.

`<C-k>sa`: the さ character.

Get help by `:h digraphs-default` or `:h digraph-table`.


### Tip 19: Overwrite Existing Text with Replace Mode

In Normal mode, `R` triger Replace mode.

In Insert mode, the `<Insert>` key in common keyboards can toggle between Insert and Replace mode. 

`gR` triggers _Virtual Replace mode_, which treats the tab character as though it consisted of spaces.

`r` `gr` provide the single-shot versions of Replace and Virtual Replace mode.

## Chap 4 - Visual Mode

### Tip 20: Grok Visual Mode

Commands that work the same as Normal mode in Visual:

- `h` `j` `k` `l` to move cursor
- `f{char}` to jump to a character 
- `;` `,` to repeat or reverse the jump by `f{char}`
- search commands (together with `n`/`N`) to jump to pattern matches
- `c` to change text and go into Insert mode (after selection)

### Tip 21: Define a Visual Selection

__Enable Visual mode from Normal mode__

| Command | Effect                             |
|---------|------------------------------------|
| `v`     | enable character-wise Visual mode  |
| `V`     | enable line-wise Visual mode       |
| `<C-v>` | enable block-wise Visual mode      |
| `gv`    | reselect the last visual selection |

__Switching between Visual modes__

| Command         | Effect                                  |
|-----------------|-----------------------------------------|
| `<Esc>`/`<C-[>` | switch to Normal mode                   |
| `v`/`V`/`<C-v>` | switch to Normal mode (when used from character-, line-, or block-wise Visual mode, respectively |
| `v`             | switch to character-wise Visual mode    |
| `V`             | switch to line-wise Visual mode         |
| `<C-v>`         | switch to block-wise Visual mode        |
| `o`             | go to the other end of highlighted text |

### Tip 22: Repeat Line-Wise Visual Commands

To make the `<` `>` commands work properly:

```
:set shiftwidth=4 softtabstop=4 expandtab
```

`Vj`: select two lines

`>.`/`2>`: indent the selected lines twice. If over shoot, `u` can undo it.

### Tip 23: Prefer Operators to Visual Commands Where Possible

Suppose we have the 3 html lines below, and would like to make the 3 words inside the tags uppercase:


```html
<a href="#">one</a>
<a href="#">two</a>
<a href="#">three</a>
```

__Using a Visual Operator__

1. `vit` to _visually_ select _inside_ the _tag_;
2. `U` to convert the selected characters to uppercase;
3. `j.` to repeat the above operations in the next line.

However, this set of commands will give such results:

```html
<a href="#">ONE</a>
<a href="#">TWO</a>
<a href="#">THRee</a>
```

A repeated Visual command affects the same range of text (in this case, only three characters).

__Using a Normal Operator__

`gUit` `j.` `j.` give good results:

```html
<a href="#">ONE</a>
<a href="#">TWO</a>
<a href="#">THREE</a>
```

### Tip 24: Edit Tabular Data with Visual-Block Mode

| Keystrokes | Meanings                                          |
|------------|---------------------------------------------------|
| `<C-v>3j`  | select a vertical column in 4 lines               |
| `x...`     | delete that column; delete 3 more columns         |
| `gv`       | reselect the last visually selected column        |
| `r|`       | replace each character in that column with `|`    |
| `yyp`      | duplicate a line                                  |
| `Vr-`      | trigger line-wise Visual mode, and replace every character in that line with `-` |

### Tip 25: Change Columns of Text

Use Visual-Block mode to insert text into several lines simultaneously.

| Keystrokes | Meanings                                                    |
|------------|-------------------------------------------------------------|
| `<C-v>jje` | trigger Visual-Block mode, select blocks across three lines, whose width decided by the word in the 3rd line |
| `c`        | delete the selected text; into Insert mode                  |
| {text}     | insert {text}; only changes of the topmost line can be seen |
| `<Esc>`    | back to Normal mode, changes in all lines can be seen       |

### Tip 26: Append After a Ragged Visual Block

Visual-Block mode is not limited to _rectangular_ regions.

| Keystrokes | Meanings                                                                     |
|------------|------------------------------------------------------------------------------|
| `<C-v>jj$` | trigger Visual-Block mode; select across 3 lines, all until the end of lines |
| `A;`       | append `;` in the end; only changes in the topmost line can be seen          |
| `<Esc>`    | back to Normal mode, changes in all lines can be seen                        |

Note: in Visual-Block mode, `I` or `A` places the cursor at the start or end of the _selection_.

## Chap 5 - Command-Line Mode

Vim traces its ancestry back to vi; vi traces its ancestry back to a line editor called ex, which is why we have Ex commands.

### Tip 27: Meet Command Line

`:` to get into Command-Line mode.

| Command                                       | Effect                                                                          |
|-----------------------------------------------|---------------------------------------------------------------------------------|
| :[range]delete [x]                            | delete specified lines [into register x]                                        |
| :[range]yank [x]                              | yank specified lines [into register x]                                          |
| :[line]put [x]                                | put the text from register x after the specified line                           |
| :[range]copy {address}                        | copy the specified lines to below the line specified by {address}               |
| :[range]move {address}                        | move the specified lines to below the line specified by {address}               |
| :[range]join                                  | join the specified lines                                                        |
| :[range]normal {commands}                     | execute Normal mode {commands} on each specified line                           |
| :[range]substitute/{pattern}/{string}/[flags] | replace occurrences of {pattern} with {string} on each specified line           |
| :[range]global/{pattern}/[cmd]                | execute the Ex command [cmd] on all specified lines where the {pattern} matches |

`<C-v>` `<C-k>` `<C-r>{register}` work in Command-Line mode like in Insert mode.

### Tip 28: Execute a Command on One or More Consecutive Lines

`:1`: specify line 1

`:$`: specify the last line

`:2,5p`: specify line 2 to line 5; and print them

`:.`: specify the current line

`:.,$p`: specify from the current line to the last line; and print them

`:%`: specify the all lines in the current file

`:%s/Pratical/Pragmatic/`: replace the first occurrence of "Practical" with "Pragmatic" on each line

We can also use visaul selection to specify lines.

`:/<html>/,/<\/html>/`: specify the range of lines beginning with a line containing `<html>` and ending with a line containing `</html>`.

`:{address}+n`: specify by the address adding with an offset.

`:/<html>/+1,/<\/html>/-1`: specify the range of lines, whose beginning line is one line below the line containing `<html>`, and whose ending line is one line above the line containing `</html>`.

__Special addresses__:

| Symbol | Address                                       |
|--------|-----------------------------------------------|
| 1      | first line of the file                        |
| $      | last line of the file                         |
| 0      | virtual line above the first line of the file |
| .      | line where the cursor is placed               |
| `'m`   | line containing mark `m`                      |
| `'<`   | start of visual selection                     |
| `'>`   | end of visual selection                       |
| %      | the entire file (shorthand for `:1,$`)        |


Here line 0 is useful in the `:copy {address}` or `:move {address}` commands when we want to copy or move a range of lines to the top of a file.

### Tip 29: Duplicate or Move Lines Using `:t` and `:m` Commands

`:copy` = `:co` = `:t`. Think of it as _copy TO_.

| Command  | Effect                                                    |
|----------|-----------------------------------------------------------|
| :t6      | copy the current line to just below line 6                |
| :t.      | duplicate the current line (similar to Normal mode `yyp`  |
| :t$      | copy the current line to the end of the file              |
| `:'<,'>t0` | copy the visually selected lines to the start of the file |

`:move` = `:m`. It functions analogously to `:t`, but in 'move' rather than 'copy' way.

### Tip 30: Run Normal Mode Command Across a Range

Use `:normal` to run Normal mode command on a range of lines.

`:'<,'>normal .`: for each line in the visual selection, execute the Normal mode dot command.

`:%normal A;`: append a `;` at the end of every line of the file.

`:%normal i//`: add `//` in the beginning of every line of the file.

### Tip 31: Repeat the Last Ex Command 

`@:`: repeat the last ex command.

### Tip 32: Tab-Complete Ex Commands

`<C-d>`: in Command mode, to reveal a list of possible completions

`<Tab>`: cycle through the possible completions

Customize the completion behaviors with the 'wildmode' option. `:h 'wildmode'` to get help.

### Tip 33: Insert the Current Word at the Command Prompt

`<C-r><C-w>` copies the word under the cursor and inserts it at the command line prompt.

### Tip 34: Recall Commands from History

Use up arrow key to get the previous command; down arrow in the opposite direction.

Arrow keys can be avoided for recalling command history, by custom mappings:

```
cnoremap <C-p> <Up>
cnoremap <C-n> <Down>
```

`:write | !python %`: join the commands `:write` and `:!python %`

`q:` in Normal mode to meet the Command-Line window.

| Command | Action                                                   |
|---------|----------------------------------------------------------|
| `q/`    | open the command-line window with history of searches    |
| `q:`    | open the command-line window with history of Ex commands |
| `<C-f>` | switch from Command-Line mode to the command-line window |

### Tip 35: Run Command in the Shell 

`:!python %`: execute the current file using python

`:shell`: hang the current Vim session, into a shell interface

`$ exit`: exit the shell and back to Vim

If we already run Vim in Bash, `<C-z>` in Vim and `fg` in Bash achieve the same function as `:shell` in Vim and `exit` in shell.

The position of `!` matters:

- `:write !sh`: pass the contents of the buffer as input to the external `sh`
- `:write ! sh`: same as above
- `:write! sh`: write the contents of the buffer to a file called `sh` by calling the `:write!` command.

__Filtering the Contents Through an External Command__

Suppose we have a csv file:

```
first name,last name,email
john,smith,john@example.com
drew,neil,drew@vimcasts.org
jane,doe,jane@example.com
```

We can sort the namelist by the last name using this command:

`:2,$!sort -t',' -k2`

Here `:2,$` specify the line range; `sort` is the external filtering command; the `-t','` option assigns that fields are separated with commas, and the `k2` flag indicates that the second field is to be used for the sort.

| Command                | Effect                                                                       |
|------------------------|------------------------------------------------------------------------------|
| `:shell`               | start a shell (return to Vim by typing `exit`                                |
| `:!{cmd}`              | execute `{cmd}` with the shell                                               |
| `:read !{cmd}`         | execute `{cmd}` in the shell and insert its standard output below the cursor |
| `:[range]write !{cmd}` | execute `{cmd}` in shell with `[range]` lines as standard input              |
| `:[range]!{filter}`    | filter the specified `[range]` through external program `{filter}`           |

