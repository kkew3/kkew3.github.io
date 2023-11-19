---
layout: post
title:  "Set up GitHub Pages on macOS"
date:   2022-07-24 18:20:49 +0800
tags:   misc
---

The steps are organized in a shell script like form:

```bash
brew install chruby
# add '. /usr/local/opt/chruby/share/chruby/chruby.sh' to .bashrc or .zshrc

# install ruby alternative to system's
brew install automake bison openssl readline libyaml gdbm libffi
curl --remote-name https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.0.tar.xz
tar xf ruby-3.1.0.tar.xz
cd ruby-3.1.0
./configure --prefix="$HOME/.rubies/ruby-3.1.0" --with-opt-dir="$(brew --prefix openssl):$(brew --prefix readline):$(brew --prefix libyaml):$(brew --prefix gdbm):$(brew --prefix libffi)"
make -j4
make install

# restart shell

# set default ruby
chruby ruby-3.1.0
# now ensure 'command -v ruby' or 'command -v gem' returns the one under ~/.rubies

# install Bundler and Jekyll
gem install bundler jekyll

mkdir /path/to/website/local/dir
cd $_
git init username.github.io
cd $_
jekyll new --skip-bundle .
# follow instruction from https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll
bundle install

# according to https://stackoverflow.com/a/70916831/7881370
bundle add webrick

# set up mathjax etc.
# follow https://github.com/jeffreytse/jekyll-spaceship#installation
# but in Gemfile, add `gem "jekyll-spaceship", "~> 0.9.9"` instead of
# using the latest version by not specifying version (see Issue #81 of
# 'jeffreytse/jekyll-spaceship' at GitHub)

bundle install

# Mathjax can now be rendered locally, but not on GitHub. That's because
# jekyll-spaceship is not in its whitelist. See
# https://github.com/marketplace/actions/jekyll-deploy-action for detail.
# Follow its instruction (including adding the github workflow file,
# creating 'gh-pages' orphan branch). Then ensure the GitHub Personal
# Access Token (PAT) has sufficient permission (for workflow specifically).
# Push master to GitHub.

# Mathjax should already be ready.

#####################################################
# The only command needed to run over and over again:
#####################################################

# build and serve locally
bundle exec jekyll serve
```
