---
layout: post
title:  "Synchronize files using git over ssh"
date:   2025-05-05 11:48:55 +0800
tags:   git
---

I typically write model code on a consumer-grade laptop, and `rsync` the code to a remote server, where the model is trained.
The price is that for every project, I have to write a script doing this job.
Furthermore, `rsync` is not very smart in managing deleted files---it either deletes all extraneous files or keep all those files.
While it's probably not impossible to overcome the limitation by carefully reading the [manual](https://linux.die.net/man/1/rsync), delving into it would take tons of time as the manual contains over 19k words.

Recently, I find that `git` over ssh can be used to resolve the isssue.
The procedure is simple.
Let us denote the remote server by `remote.server`, i.e. with an entry in `~/.ssh/config` similar to:

```sshconfig
Host remote.server
  User ubuntu
  HostName XXX.XXX.XXX.XXX
  IdentityFile ~/.ssh/id_rsa
```

On the local laptop:

```bash
cd /path/to/my_project
# We store the bare repo under ~/git/my_project.git.
ssh remote.server 'git init --bare ~/git/my_project.git'
# Setup remote over ssh.
git remote add origin remote.server:~/git/my_project.git
git push -u origin master
```

On the remote server (if we are setting up from scratch):

```bash
cd /path/to/projects/dir
# Git clone local repo.
git clone ~/git/my_project.git
```

On the remote server (if we already have an existing remote directory):

```bash
cd /path/to/my_project
tmp=$(mktemp -d tmpXXXXXX)
# Git clone local repo.
git clone ~/git/my_project.git $tmp
# See https://stackoverflow.com/a/13852329/7881370.
mv $tmp/.git .git
rm -rf $tmp
# Then resolve conflicts by referring to git status.
#git status
```

After this setup, for some further local updates, we just `git push` locally and `git pull` remotely.
