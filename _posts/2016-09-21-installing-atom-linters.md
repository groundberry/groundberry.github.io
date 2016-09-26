---
layout: post
title:  "Installing Atom linters"
date:   2016-09-21
categories: setup
---

A linter is a tool that goes through our code and identifies potential problems before we've even had the chance to run it.

## Installing *linter-ruby*

The first linter we are going to install is `linter-ruby`. It's pretty basic (it just runs our source through `ruby -wc`), but it can be useful to catch obvious syntax errors.

To install it, launch Atom, and click on the *Atom > Preferences* menu item (or press `Cmd + ,`). Navigate to the *Install* section, and search for `linter-ruby`. Now you just need to press the *Install* button:

{% include image.html name="installing_linter_ruby.png" alt="Installing linter-ruby" %}

We can see the linter in action here, as I forgot an `end`:

{% include image.html name="linter_ruby.png" alt="Installing linter-ruby" %}

We didn't need to execute our program to find the error. The linter caught it immediately!

## Installing *linter-rubocop*

The second linter we are going to install is `linter-rubocop`. It enforces many of the guidelines outlined in the [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide), which I've found helpful in improving the readability of my code and preventing mistakes.

To install it, follow the same steps as with `linter-ruby`:

{% include image.html name="installing_linter_rubocop.png" alt="Installing linter-rubocop" %}

Here the linter suggests turning a single-line `if` statement into a guard clause, to avoid unnecessary nesting:

{% include image.html name="linter_rubocop.png" alt="Installing linter-rubocop" %}

These suggestions help us internalise the guidelines, so that we continuously improve our code.
