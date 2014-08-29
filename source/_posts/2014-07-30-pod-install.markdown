---
layout: post
title: "Speed up pod install"
date: 2014-07-30 22:30:54 -0600
comments: true
categories: cocoapods shell
---

## pod install

If you use CocoaPods, you're familiar with the `pod install` command. Especially in the process of creating a pod, you end up running this command a lot. The annoying part is that there's a delay before it starts returning much output. This is where the pod gem reaches out to check to see if the specs have changed at all.

Even when the pod version specifiers haven't changed, CocoaPods reaches out to fetch any updates to the `master` spec repo, along with any private spec repos that may be configured. During local development of a pod, this slows down the process unnecessarily as the only changes you care about are local to your hard drive.

<!-- more -->

## --no-repo-update

The **--no-repo-update** switch suppresses this spec repo update and speeds up the install command considerably.

Since that is way too much to type for such a common command to fly across my terminal, I like to wrap these up in a nice, short alias.

```
alias pi='pod install --no-repo-update'
```

Now, you don't want to use this alias every time you need to do a `pod install` as your `master` spec repo would get out of date and you could miss important updates.



