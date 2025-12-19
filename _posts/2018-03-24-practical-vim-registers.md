---
layout: post
title: "Practical Vim: Registers"
date: 2018-03-24 01:00:00
categories: en
tags: Vim
---

> Series notes on _Practical Vim_ by D. Neil:
> - [Practical Vim: Modes](/2018/03/17/practical-vim-modes/)
> - [Practical Vim: Files](/2018/03/19/practical-vim-files)
> - [Practical Vim: Getting Around Faster](/2018/03/20/practical-vim-getting-around-faster)
> - Practical Vim: Registers

__Contents__

* content
{:toc}

## Chap 10 - Copy and Paste

### Tip 59: Delete, Yank, and Put with Unnamed Register

Delete/Yank without specifying a register will put the characters into the unnamed register.

Paste(_Put_) without specifying a register will put the characters from the unnamed register.

`y{motion}` for yank with motion.

`x` for deleting a character.

`s` for deleting a character and then into Insert mode.

`d{motion}` for deleting with motion.

`p` for pasting.

### Tip 60: Grok Registers

`"{reg}`: address a register.

`"bdd`: cut the current line into register `b`.

`"ayiw`: yank the current word into register `a`.

`""` is the unnamed register.

`"0` is the Yank register, only for `y{motion}`.

`"a`-`"z` are the Named Registers for users to explicitly use.

`"_` is the Black Hole Register, 'eating' anything put into it.
`"_d{motion}` will delete the specified text without saving a copy of it.

`"+` is the System Register (The X11 clipboard, used with cut, copy and paste).

`"*` is the Selection Register (The X11 primary, used with middle mouse button).

`"=` is the Expression Register, check Tip 16.

More Registers:

| Register | Contents                     |
|----------|------------------------------|
| `"%`     | name of the current file     |
| `"#`     | name of the alternative file |
| `".`     | last insert text             |
| `":`     | last Ex command              |
| `"/`     | last search pattern          |


### Tip 61: Replace a Visual Selection with Register

On selecting some text, _Put_ will replace it with the text in the registers.
In this case, the replaced text would be put in the register (think of it as being deleted by `d`).
This technique can be use to swap two words.

### Tip 62: Paste from a Register

`p` `P` in Normal mode, `<C-r>` in Insert mode.

`gp` `gP` paste the same thing,
but leaving the cursor at the end of pasted text instead of at the beginning.

### Tip 63: Interact with the System Clipboard

In Insert mode, `Ctrl-Shift-v` or `Ctrl-Alt-v` provided by the terminal,
or `<C-r>+` may cause wrong indentation if `autoindent` is enabled.
The `paste` option can help with this.
When it is enabled, Vim turns off all Insert mode mappings and abbrevations and resets a host of options,
including `autoindent` (look up `:h 'paste'`).

`"+p` in Normal mode is better.

## Chap 11 - Macros

### Tip 64: Record and Execute a Macro

`q{register}` to begin recording a macro into a given register; `q` again to stop recording.

`@{register}` to execute the recorded contents in the specified register.

`@@` repeats the macro that was invoked most recently.

### Tip 65: Normalize, Strike, Abort

When executing a macro, Vim blindly repeats the sequence of canned keystrokes.
Therefore,

1) Make sure the cursor is explicitly positioned where you expect (like `0`,`gg`, etc.).

2) Strike the target with a repeatable motion: word-wise motion is better than character-wise;
navigating by search can be exploited.

3) Abort when a motion fails - by default Vim aborts the rest of the macro if a motion fails,
so we need not worry `100@a` may over-count,
and can use a very large number if we want to repeat for many times.

### Tip 66: Play Back with a Count

`11@{register}`: execute a macro in {register} for 11 times.

Again, since Vim would abort the execution if a motion fails,
so you can provide a large enough count without much worry.

### Tip 67: Repeat a Change on Contiguous Lines

Record a series of operations on a line into a macro in `{register}`, with the last keystroke being `j`.
Then `5@{register}` would repeat the same change to the next 5 contiguous lines in series.

One problem of executing macro in series: if the executation fails in one line in the middle,
it would be aborted, leaving the following lines unchanged.

Executing in parallel is a better choice in this case: use `V` to trigger line-wise Visual mode,
select those lines, and `:'<,'>normal @{register}` to execute the macro on each of the lines.

### Tip 68: Append Commands to a Macro

`qA` will record keystrokes, _appending_ them to the existing contents of register `a`.

### Tip 69: Act Upon a Collection of Files

1. Build a list of target files with `:args`, e.g.

   ```
   :args *.py
   ```

2. Make sure we are at the start of the argument list:

   ```
   :first
   ```

3. Now record a macro to reg `a`, while applying a series of operations.

4. Revert the change we have just made to the first buffer, to prevent applying the change twice on it
(because we would execute the macro to ALL buffers in the argument list later):

   ```
   :edit!
   ```

5. Execute the macro on all of the buffers in the argument list:

   ```
   :argdo normal @a
   ```

   This macro can also be executed in series across many files,
   by appending a final step that advances to the next buffer in the list
   (analogous to the `j` in the macro applied on contiguous lines) :

   | Keystrokes | Effect                                          |
   |------------|-------------------------------------------------|
   | `qA`       | Begin recording new operations to append to `a` |
   | `:next`    | Advance to next buffer                          |
   | `q`        | Stop recording                                  |
   | `22@a`     | Execute the macro on many buffers               |

6. Save changes to all files:

   ```
   :wall
   ```

### Tip 70: Evaluate an Iterator to Number Items in a List

Suppose we have this text:

```
partridge in a pear tree
turtle doves
French hens
calling birds
golden rings
```

We want it to be:

```
1) partridge in a pear tree
2) turtle doves
3) French hens
4) calling birds
5) golden rings
```

For this task we use a variable and increment it, which would look like:

```
:let i=0
:let i += 1
```

Specifically, 1) record the macro:

| Keystrokes               | Effect                                                 |
|--------------------------|--------------------------------------------------------|
| `:let i=1`               | declare the variable                                   |
| `qa`                     | begin recording                                        |
| `I<C-r>=`i`<CR>`)`<Esc>` | insert to the line beginning, getting the value of `i` |
| `:let i += 1`            | increment `i`                                          |
| `q`                      | stop recording                                         |

2) execute the macro:

| Keystrokes        | Effect                         |
|-------------------|--------------------------------|
| `jVG`             | select all the following lines |
| `:'<,'>normal @a` | execute the macro in parallel  |

### Tip 71: Edit the Contents of a Macro

The contents of a macro in a register are the very same with which the Yank/Put operations interact.
So we can paste the macro contents as plain text and edit them, 
and then yank the contents back to the register.

Moreover, we can manipulate the text of a macro programmatically using Vim script, like

```
:let @a=substitute(@a, '\~', 'vU', 'g')
```

Look up `:h substitute()` for details.
