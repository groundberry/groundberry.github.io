---
layout: post
title:  "Fixing permissions of files and directories"
date:   2019-03-02
categories: development
published: true
---

Every time I copy files from an NTFS-formatted drive (one that was formatted from Windows) into macOS, their permissions are messed up. If they happen to be part of a Git repo, they'll all show up as modified, with diffs looking like this:

```
$ git diff
diff --git a/.gitignore b/.gitignore
old mode 100644
new mode 100755
diff --git a/Gemfile b/Gemfile
old mode 100644
new mode 100755
diff --git a/Gemfile.lock b/Gemfile.lock
old mode 100644
new mode 100755
...
```

The quickest way I've found of fixing this is by using `find` with its `-type` and `-exec` flags, like this:

```
$ find . -type f -exec chmod 644 {} \;
```

This command will find all files under the current path (`.`) and execute the command `chmod 644 <file>` on each of them.

We can also fix directories with this similar command:

```
$ find . -type d -exec chmod 755 {} \;
```

This will find all directories in the current path (`.`) and execute the command `chmod 755 <directory>` on each of them.

Now our `git diff` should go back to normal. Phew!