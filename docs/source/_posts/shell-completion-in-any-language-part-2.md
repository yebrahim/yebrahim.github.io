---
title: shell completion in any language - part 2
date: 2021-04-11 11:01:51
tags:
---

Part one here: {% post_link shell-completion-in-any-language %}

Now that we have the skeleton, let's flesh it out and add the completion
logic. The basic idea is we pass some environment variables from the shell
bridge script, read those in the completer and use them to suggest
completions.

In the shell bridge, we have this part for zsh for example:
```zsh
  reply=($(COMP_CWORD="$((CURRENT-1))" COMP_LINE="$BUFFER" _silly_completer.py -- "${words[@]}"))
```

The `BUFFER` variable is populated by zsh (see full spec [here](http://zsh.sourceforge.net/Guide/zshguide06.html)), containing
the the entire line respectively the user requested auto-complete for.

Similarly, bash has `COMP_WORDS`, `COMP_LINE`, and a few others too, you can
read more on these in the [spec](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html) as well.

We pass this information to the completer, which can now parse and respond,
so let's get to the fun part.

We started with a Python script for the `silly` cli, let's fill it out here.
The first part is to parse and get the line passed to it from the shell:

```python
words = os.environ.get('COMP_LINE').split()
```

If `words` contains a single word, it must be our program name, since this is
why our completer would be called in the first place. If this is the case, we
should return the list of subcommands if any. Returning works by just
printing to stdout. Something like:

```python
if len(words) < 2:
  # top level
  for c in ['subcmd1', 'subcmd2', '--help', '-h']:
    print(c)
```

Now if you type `silly ` then press the double Tab, you should see these
four options suggested. Let's go a level deeper and say that `subcmd1` should
have three options, we can test it this way:

```python
else: # if not top level
  if words[1] == 'subcmd1':
    for c in ['op1', 'op2', 'op3']:
      print(c)
```

That's about it.
