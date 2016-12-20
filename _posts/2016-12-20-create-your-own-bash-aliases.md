---
layout: post
title:  "Create your own Bash aliases"
date:   2016-12-20
categories: development
---

A Bash alias is nothing more than an abbreviation or shortcut. When we are working in a terminal, we tend to type the same commands over and over. In order to avoid typing long commands, we can use aliases to replace them with just a few letters. In the next sections I'll show you how to create aliases, and some examples that I personally use.

## How to create a Bash alias

We can declare temporal aliases that last as long as our shell session by using the `alias` command:

```
$ alias alias_name="command_to_run"
```

If we want to remove the alias we have just created, we can use `unalias`:

```
$ unalias alias_name
```

We can list all declared aliases by running `alias` without arguments.

If we want to use our aliases in every shell session, we can add them to one of the files that is read every time we open our terminal, such as `~/.bashrc` or `~/.bash_profile`.

## Some examples

Here are some of the aliases that I use.

### List all files in long format

```
alias ll="ls -al"
```

### Create intermediate directories by default

```
alias mkdir="mkdir -p"
```

### Go up the directory tree quickly

```
alias ..="cd .."
alias ...='cd ../../../'
alias ....='cd ../../../../'
```

Now go and think of the commands you use most often that would benefit from an alias. 🤔
