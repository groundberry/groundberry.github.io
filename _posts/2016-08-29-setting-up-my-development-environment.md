---
layout: post
title:  "Setting up my development environment"
date:   2016-08-29 14:46:22 +0100
categories: setup
---

Today is the first day of the pre-course at Makers Academy. During the previous weeks I have been setting up my laptop in order to start learning and getting familiar with the command line, Git, GitHub and Ruby.

To prepare my development environment on my MacBook I followed these steps:

## Xcode

We'll install [Xcode](https://developer.apple.com/xcode/) from the App Store. We need this because it includes `git` and other useful development tools.

## Homebrew

We'll install [Homebrew](http://brew.sh) following the instruction on their page. It will help us install other tools we'll need below.

## Editor

As our code editor we'll install either [Atom](https://atom.io/) or [Visual Studio Code](https://code.visualstudio.com) using Homebrew (we use `brew cask` because it's a GUI app):

```bash
$ brew cask install atom # install atom...
$ brew cask install visual-studio-code # ... or visual studio code
```

## Node

To manage our [Node](https://nodejs.org) versions, we'll install `nvm` using Homebrew (we use just `brew` because it's a CLI app):

```bash
$ brew install nvm # install nvm
$ nvm install 6.3.1 # install node version 6.3.1
$ nvm alias default 6.3.1 # set 6.3.1 as the default version
```

And add the following lines to our `.bashrc` file so that `nvm` is initialised properly every time we open a terminal:

```bash
export NVM_DIR="$HOME/.nvm"
source "$(brew --prefix nvm)/nvm.sh"
```

We check that everything is working:

```bash
$ node --version # check node version
v6.3.1
$ npm --version # check npm version
3.10.3
```

## Ruby

To manage our [Ruby](https://www.ruby-lang.org) versions, we'll install `rbenv` and `ruby-build` using Homebrew (we use just `brew` because it's a CLI app):

```bash
$ brew install rbenv ruby-build # install rbenv
$ rbenv install 2.3.1 # install ruby version 2.3.1
$ rbenv global 2.3.1 # set 2.3.1 as the default version
```

And add the following lines to our `.bashrc` file so that `rbenv` is initialised properly every time we open a terminal:

```bash
export RBENV_DIR="$HOME/.rbenv"
eval "$(rbenv init -)"
```

We check that everything is working:

```bash
$ ruby --version # check ruby version
v2.3.1
```

## Git

We'll create a `.gitconfig` file in our home folder:

```bash
$ touch ~/.gitconfig
```

We can add our name and email address to the file so that this information is added to all of our [Git](https://git-scm.com/) commits:

```
[user]
  name = Blanca Mendizabal Perello
  email = blanca.mendizabal.perello@gmail.com
```

Optionally, we can configure Git to use our editor of choice (instead of the default, which is `vi`) by adding these two lines to our `.gitconfig` file:

```
[core]
  editor = atom --wait
```

*[CLI]: Command Line Interface
*[GUI]: Graphical User Interface
