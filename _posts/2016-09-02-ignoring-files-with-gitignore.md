---
layout: post
title:  "Ignoring files with .gitignore"
date:   2016-09-02
categories: setup
---

When we are working inside our Git repository and we run `git add .` in order to prepare things for a commit, sometimes we'll see unwanted files being staged and we'll have to manually remove them by running `git rm --cached <filename>`. But there's a better way! We can tell Git to automatically ignore certain files or directories by using a [`.gitignore`](https://git-scm.com/docs/gitignore) file.

We can create local or global `.gitignore` files.

## Local *.gitignore* file

We can create a `.gitignore` file in our project's root folder to ignore items that are specific to this project. For example, when I'm working on [my blog using Jekyll]({% post_url 2016-08-30-creating-my-blog-on-github-pages %}), a folder named `_site` containing all the generated HTML will be created. To prevent it from being committed, we can create a local `.gitignore` file like this:

```bash
$ cd myblog
$ echo "_site/" > .gitignore
```

That way the `_site` folder will never be committed by mistake.

## Global *.gitignore* file

When we navigate through folders using Mac's Finder it will generate hidden [`.DS_Store`](https://en.wikipedia.org/wiki/.DS_Store) files all over the place. Instead of adding `.DS_Store` to all the different `.gitignore` files in our various projects, we can create a global `.gitignore` like this:

```bash
$ cd ~
$ echo ".DS_Store" > .gitignore
```

Git doesn't automatically pick up this global `.gitignore` file. We need to add it to our `.gitconfig`. Although we can do it manually by editing the file with our favourite text editor, it may be easier to just run the following command in our terminal:

```bash
$ git config --global core.excludesfile "~/.gitignore"
```

We'll never see a `.DS_Store` file again!
