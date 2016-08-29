---
layout: post
title:  "Creating my blog on GitHub Pages"
date:   2016-08-30 14:46:22 +0100
categories: setup
---

I have decided to create my blog using [GitHub Pages](https://pages.github.com/) and [Jekyll](https://jekyllrb.com/), to have more control over the whole publishing process while practising with the command line and Git at the same time.

I followed these steps:

## GitHub

We will create a new repository on GitHub named `username.github.io`, where `username` is your username on GitHub (in my case, `groundberry.github.io`).

## Jekyll

Assuming we already have Ruby set up on our machine (see [Setting up my development environment]({% post_url 2016-08-29-setting-up-my-development-environment %})), we will install [Jekyll](https://jekyllrb.com/) and [Bundler](http://bundler.io/) in one go, and we will create a new Jekyll project with the same name as the repo we just created:

```bash
$ gem install jekyll bundler # install jekyll and bundler
$ jekyll new groundberry.github.io # create new jekyll project
``` 

We use Bundler to install the project's dependencies:

```bash
$ cd groundberry.github.io
$ bundle install # install dependencies
```

Now we execute Jekyll locally to view our site:

```bash
$ bundle exec jekyll serve # serve generated site
```

Go to <http://localhost:4000> using your browser to check out the results.

## Publishing

We will initialise a Git repository inside the folder containing our Jekyll project, and add a remote pointing to the repository we created previously on GitHub:

```bash
$ git init # init repo
$ git remote add origin https://github.com/groundberry/groundberry.github.io.git # add remote
```

Now we will commit all our changes and push them to the remote:

```bash
$ git add . # stage changes
$ git commit -m 'Initial commit' # commit changes
$ git push --set-upstream origin master # push changes and set a default remote
```

Finally, accessing <https://groundberry.github.io> we will see the website published. Any changes we commit and push to the remote from now on will cause the website to be automatically updated.