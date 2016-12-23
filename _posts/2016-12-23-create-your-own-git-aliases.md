---
layout: post
title:  "Create your own Git aliases"
date:   2016-12-23
categories: development
---

Since I wrote my previous post [Create your own Bash aliases]({% post_url 2016-12-20-create-your-own-bash-aliases %}) I've learnt that I can also create aliases for Git. During these months learning to code I've repeated the same Git commands so many times... Often the commands are so long and cryptic that I'm never sure whether I'm writing them correctly, or whether I will delete all my staged and unstaged changes forever. Git aliases can help us solve both issues.

## How to create a Git alias

A Git alias is a shortcut. We can do the same task just typing `git` followed by the alias we have defined. Easy!

We are going to create a new alias so that we can type `co` instead of `checkout`. We can do so by running:

```
$ git config --global alias.co checkout
```

This will add a new entry in the `.gitconfig` file in our home directory, under the `[alias]` section.

```
$ cat ~/.gitconfig

[alias]
  co = checkout
```

We can also edit the file manually, and skip running the command.

## Some examples

Here are some other aliases that I personally use and find really helpful.

### Short status

```
s = status --branch --short --untracked=no
```

```
$ git s

## master
 M tmp.txt
```

### Long status

```
ss = status
```

```
$ git ss

On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   tmp.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

### Log in graph format, showing the last 20 commits

```
l = log --graph --max-count=20 --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%an, %cr)%Creset'
```

```
$ git l

* 3fe6a5d - (HEAD -> master, origin/master, origin/HEAD) Privatize unneededly protected methods in Active Support tests (Akira Matsuda, 29 minutes ago)
* 10fb721 - Privatize unneededly protected methods in Railties tests (Akira Matsuda, 29 minutes ago)
```

### Log in graph format, showing all commits

```
ll = log --graph --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%an, %cr)%Creset'
```

```
$ git ll

*   fd63aa0 - Merge pull request #27424 from utilum/fix_complex_and_rational_are_duplicable (Kasper Timm Hansen, 20 hours ago)
|\  
| * 83d3b0d - Fix Complex and Rational are duplicable? (utilum, 2 days ago)
* |   2fd3034 - Merge pull request #27431 from y-yagi/quiet_generator_log_in_test (Kasper Timm Hansen, 33 hours ago)
|\ \  
| * | baa8c5a - quiet generators log in test (yuuji.yaginuma, 2 days ago)
* | |   3ae1035 - Merge pull request #27430 from kirs/aj-warning (Matthew Draper, 2 days ago)
|\ \ \  
| |/ /  
|/| |   
| * | 2f421fa - Remove warning in ActiveJob (Kir Shatrov, 2 days ago)
|/ /  
* |   7336d0a - Merge pull request #27427 from rails/binary-params (Aaron Patterson, 2 days ago)
|\ \  
| * | 8710562 - updating docs (Aaron Patterson, 2 days ago)
| * | 2eb0a66 - Document and update API for `skip_parameter_encoding` (Aaron Patterson, 2 days ago)
|/ /  
* | 85b2a3e - fix typo in getting_started [ci skip] (#27423) (yachibit, 2 days ago)
|/  
*   cb02920 - Merge pull request #27355 from yukideluxe/fixtures-deleted-tables (Kasper Timm Hansen, 2 days ago)
```

### Unstage changes

```
unstage = reset --
```

```
$ git unstage

Unstaged changes after reset:
M	tmp.txt
```

### Amend last commit

```
fix = commit --amend --no-edit
```

```
$ git fix

[master 32264bb] Some commit
 Date: Fri Dec 23 15:51:15 2016 +0100
 1 file changed, 1 insertion(+)
 create mode 100644 tmp.txt
```

With this basic knowledge of Git aliases you can go crazy and create your own. 😎
