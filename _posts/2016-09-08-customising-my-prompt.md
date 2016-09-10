---
layout: post
title:  "Customising my prompt"
date:   2016-09-08
categories: setup
---

By default, when I open the terminal on my Mac, it shows the following prompt:

```
macbook:~ blanca$
```

It seems to be showing the name of my computer, my current directory, my username, and a dollar.

However, if I `cd` into a folder, it only displays the folder name, not the full path to the folder.

```
macbook:~ blanca$ cd project
macbook:project blanca$
```

This makes it difficult to know where we are when we have to deal with many different files and folders.

That's why I decided to customise my prompt. It turns out it's really easy!

## How to start

When we open a terminal, we are running a program called `bash`, which is the one that prints our prompt, reads our input, and executes commands.

We can configure our prompt by modifying the `PS1` variable from `bash`. We can print its current value by running:

```
macbook:project blanca$ echo $PS1
\h:\W \u\$
```

What are those weird characters? If we look at the `bash` manual by running `man bash`, we can see an explanation under the *PROMPTING* section:

```
\a     an ASCII bell character (07)
\d     the date in "Weekday Month Date" format (e.g., "Tue May 26")
\D{format}
       the format is passed to strftime(3) and the  result  is  inserted  into  the  prompt
       string;  an  empty  format  results  in  a locale-specific time representation.  The
       braces are required
\e     an ASCII escape character (033)
\h     the hostname up to the first `.'
\H     the hostname
\j     the number of jobs currently managed by the shell
\l     the basename of the shell's terminal device name
\n     newline
\r     carriage return
\s     the name of the shell, the basename of $0 (the portion following the final slash)
\t     the current time in 24-hour HH:MM:SS format
\T     the current time in 12-hour HH:MM:SS format
\@     the current time in 12-hour am/pm format
\A     the current time in 24-hour HH:MM format
\u     the username of the current user
\v     the version of bash (e.g., 2.00)
\V     the release of bash, version + patch level (e.g., 2.00.0)
\w     the current working directory, with $HOME abbreviated with a tilde
\W     the basename of the current working directory, with $HOME abbreviated with a tilde
\!     the history number of this command
\#     the command number of this command
\$     if the effective UID is 0, a #, otherwise a $
\nnn   the character corresponding to the octal number nnn
\\     a backslash
\[     begin a sequence of non-printing characters, which could be used to embed a terminal
       control sequence into the prompt
\]     end a sequence of non-printing characters
```

## Showing the path to our working directory

According to the list above, if we want to see the full path to our current working directory, we need to use `\w` instead of `\W`. Let's try it:

```
macbook:project blanca$ PS1="\h:\w \u\$ "
macbook:~/project blanca$
```

It worked!

## Persisting our changes

If we close our terminal and open a new one, we'll see the old prompt again. Oh no, we've lost all our changes!

To make our changes permanent, we need to add them to our `.bashrc` file, so that they are run every time `bash` starts:

```
macbook:~/project blanca$ echo 'PS1="\h:\w \u\$ "' >> ~/.bashrc
```

Now that you know the basics, you can go crazy and customise it all!

**Updated:**

OMG! We can add emojis to our prompt too!

```
macbook:~ blanca$ PS1="\h:\w \u🦄 "
macbook:~ blanca🦄
```
