---
layout: post
title: "Speed up pod install"
date: 2014-07-30 22:30:54 -0600
comments: true
published: true
categories: cocoapods shell
---

If you use CocoaPods, you're familiar with the `pod install` command. Especially in the process of creating a pod, you end up running this command a lot. The annoying part is that there's a delay before it starts returning much output. This is where the pod gem reaches out to check to see if the specs have changed at all.

Even when the pod version specifiers haven't changed, CocoaPods reaches out to fetch any updates to the `master` spec repo, along with any private spec repos that may be configured. During local development of a pod, this slows down the process unnecessarily as the only changes you care about are local to your hard drive.

```
$ pod install --verbose

Analyzing dependencies

Updating spec repositories
  $ /usr/local/bin/git rev-parse  >/dev/null 2>&1
  $ /usr/local/bin/git ls-remote
  From https://github.com/CocoaPods/Specs.git
  f83cbf438a0f80b7df76534ffa08efcb359ce982	HEAD
  f83cbf438a0f80b7df76534ffa08efcb359ce982	refs/heads/master
  fe7df26fe5545072c11abac241d73087a29e87d9	refs/pull/1/head
  1748dab37fe08120775777a084e0fb9da10c4a63	refs/pull/1/merge
  ...
  5a7c7be28f0859865035c324e625c507cf858f4a  refs/pull/9998/head
  7a19ff8bfbaf00a485ca2274a229205d715dcbf7  refs/pull/9999/head
Updating spec repo `master`
  $ /usr/local/bin/git pull --ff-only
  ...yawn...
```

<!-- more -->

## --no-repo-update

The **--no-repo-update** switch suppresses this spec repo update and speeds up the install command considerably.

Since that is way too much to type for such a common command to fly across my terminal, I like to wrap these up in a nice, short [alias](https://github.com/phatblat/dotfiles/blob/master/.dotfiles/shell/alias.zsh#L31).

```
alias pi='pod install --no-repo-update'
```

Another situation where suppressing this automatic spec repo update is helpful is when you have unreliable or no network connection. The spec repo update is going to fail anyway.

Now, you don't want to use this alias every time you need to do a `pod install` as your `master` spec repo would get out of date and you could miss important updates. In general, I allow `pod install` or `pod update` to update my spec repos the first time I run them on any given day and suppress the repo update the rest of the day.
