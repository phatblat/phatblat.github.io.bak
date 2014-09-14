---
layout: post
title: "Git Sparse Checkout"
date: 2014-09-13 22:30:57 -0600
comments: true
published: true
categories: git tricks
---

Have you ever wanted to download just a subfolder of a git repo? Sure, you can clone the whole repo and then drill into that subfolder. However, if the point of just bringing down a subfolder is for easier distribution of some files, the other files can be distracting (i.e. "which sub-sub-sub folder did she say to open?").

<!-- more -->

Git is a system designed to remember every change to the tracked files in a repository. Once a file goes in, it's there for good (unless you slice and dice it with filter-branch, but that has side-effects). Since remembering your files and their content is a primary feature, git doesn't currently have the ability to clone just a subfolder (which would save bandwidth). However, git does have the ability to checkout a subfolder. This is what's referred to as a "sparse checkout".

The exercise here is to get a copy of all the images from my blog. Not terribly exciting, but I've devised a script which should be easy for you to adapt for this same purpose for any other repo.

## The Code

<script src="https://gist.github.com/phatblat/a5caa3bb3a3784f03000.js"></script>

I'm going to walk through this script with the lines a bit out of order, variables will be interspersed with the code that uses them. I generally build scripts with all the variables at the top to make it easy to see what can be tweaked.

```
tmpdir=/tmp/phatblat-images
mkdir $tmpdir
cd $tmpdir
git init
```

This sets up a fresh, empty repository in a temporary directory.

```
remote_git=git://github.com/phatblat/phatblat.github.io.git
git remote add -f origin $remote_git
```

The **-f** flag to the `git remote add` command causes git to immediately fetch from the remote when it is added. This is where this approach is less than idea. All objects in the repo are fetched from the remote, even though only a subset will be checked out. However, this keeps the repo functional as I'll talk about at the end.

## Git Protocol

The reason for the `git://` protocol is because this is the [fastest transfer protocol available](http://git-scm.com/book/en/Git-on-the-Server-The-Protocols#The-Git-Protocol). This is the best approach for fast, read-only access to a repo, but could fail from inside a corporate firewall.

```
remote_https=https://github.com/phatblat/phatblat.github.io.git
git remote add -f origin $remote_git || \
	git remote set-url origin $remote_https && \
	git fetch origin
```

If the fetch fails due to the `git://` protocol being blocked (port 9418), the indented commands will change the origin remote's url to one that uses https and try fetching again.

```
git config core.sparseCheckout true
subdir=images/
echo "$subdir" > .git/info/sparse-checkout
```

The new repo is configured to with sparse checkout enabled for only the "images" folder. No other folders will be checked out.

```
git checkout master
```

The checkout is where the filtering happens. Only the images folder is actually checked out.

```
subdir=images/
open $subdir
```

This [open](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/open.1.html) command on OS X is very handy for opening a folder in the Finder.

## Use Cases

Beyond reducing the noise in a repo, this sparse checkout has other uses. The resulting local repo is a fully functional git repo. You can browse the history and create new commits. A functional repo is especially handy for designers to manage images.

Note that if you plan to use a repo with sparse checkout for creating new commits and pushing these back to a central remote, you'll need to ditch the `git://` protocol since it can only be used for read-only access. Either HTTPS or [SSH](https://help.github.com/articles/which-remote-url-should-i-use#cloning-with-ssh) style URLs can be used for pushing changes up. Also, if you've already cloned a repo using the `git://` protocol, you don't need to re-clone it to change this protocol. Just use the `git remote set-url` command to change it.

For more details around what you can do with it, check out the docs on [sparse checkout](http://git-scm.com/docs/git-read-tree#_sparse_checkout).
