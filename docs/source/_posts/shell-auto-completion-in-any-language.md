---
title: shell auto completion in any language
date: 2021-04-11 00:34:07
tags:
---

So, you have this cli tool whose options you could never remember, or you're
a system admin with too many commands that don't have shell completion.

You think that, you know, you're a programmer, how hard could it be to write
bash/zsh/fish.. completion for your most used tool.

A few hours later, you still can't believe how hard this is. Surely there's
an easier way than reading through [bash's docs](https://www.gnu.org/software/bash/manual/bash.html#Programmable-Completion), right? Right?
Being fluent in several programming languages and having passing knowledge
about a dozen others, why can't you just write your completion function in
C++, Python, Node.js, or Go? Heck, why not write it in Elm if that's your cup
of tea?

Well, you could.

Let's get down to business. Two steps we need here:
1. An executable that will read the current line the shell is trying to
auto-complete and print suggestions. Let's call this "the completer."

2. Wiring the shell to use the completer whenever a certain program is
invoked for auto-complete.

Let's do this for a cli called `silly`.

## One: The completer

Python is readily available on most programming environments, so I'll use it
here, but the same principle applies to any executable your shell knows where
to find it and how to call it. The only requirement is you know how to parse
environment variables in your language, and how to print to stdout.

Create an executable Python script somewhere on PATH and make it executable:
```zsh
touch /usr/local/bin/_silly_completer.py
chmod +x /usr/local/bin/_silly_completer.py
```

Open the file and add these lines:
```python
#! /usr/bin/env python

print('hello')
print('world')
```

Test you've done the right thing by reloading your shell (e.g. `exec zsh`)
and running `_silly_completer.py` from any directory, make sure you see the
two lines printed to console.

That's it. We're basically saying return two suggestions: `hello` and `world`
whenever you're called, doesn't matter the input.

## Two: Wiring your shell to use the completer

Somewhere in your shell config file (`.bashrc`, `.bash_profile`, `.zshrc`...
etc), you'll want to tell it to use your completer whenever it needs to
auto-complete for a certain program.

This is a little different per shell, but the main idea is the same. I'll
include a sample for bash and zsh here since these are pretty popular, after
Apple decided zsh is the default shell on MacOS.

### Bash
```bash
_silly () {
  local words
  if type _get_comp_words_by_ref &>/dev/null; then
    _get_comp_words_by_ref -n = -n @ -n : -w words
  else
    words=("${COMP_WORDS[@]}")
  fi
  COMPREPLY=($(COMP_LINE="$COMP_LINE" _silly_completer.py -- "${words[@]}" 2>/dev/null))
}
complete -o default -F _silly silly
```

### Zsh
```zsh
_silly () {
  reply=($(COMP_CWORD="$((CURRENT-1))" COMP_LINE="$BUFFER" _silly_completer.py -- "${words[@]}"))
  _describe 'values' reply
}
compdef _silly silly
```

Reload your shell again, type `silly ` (add a space after) then press Tab
twice, you should see `hello` and `world` suggested. If you type `silly he`
and press Tab, the shell should auto-complete it to `silly hello`.

The reason we had to press a space before we got auto-completion is because
`silly` doesn't (probably) actually exist on your system.

_Happy completions!_

## Caveats

Note that the shell will run your completer executable every time it's asked
to auto-complete your command. Some shell setups like zsh plugins can try to
auto-complete after every character you type (after the initial program name,
`silly`), so make sure the script is quick enough to run that you won't feel
it running as you're typing your command.
