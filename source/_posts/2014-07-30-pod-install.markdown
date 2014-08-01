---
layout: post
title: "Speed up pod install"
date: 2014-07-30 22:30:54 -0600
comments: true
categories: cocoapods shell
---

## pod install

If you have created a podspec or two, you've probably ran `pod install` alot. Even those that only run it occasionally may have noticed that there's a delay before it starts returning much output.

Even when you haven't changed any version specifiers for your dependencies, CocoaPods reaches out to fetch any changes to the `master` spec repo. During local development of a pod, this slows down the process unnecessarily as the only changes you care about are local to your hard drive.

<!-- more -->

## --no-repo-update

The `pod install --help` command shows a few options, including **--no-repo-update** which suppresses the spec repo update and speeds up the install command.

Since that is way too much to type for a very common command.

```
alias pi='pod install --no-repo-update'
```

Obviously, you don't want to use this alias every time you need to do a `pod install` as your `master` spec repo would get out of date.

## Local :path vs. Remote Spec Repo

This works great for pods you're working on which are specified via local :path in the `Podfile`.

```
pod 'Fetchable', :path => '../pods/Fetchable'
```

But, what about versions of the pod you want to test via a spec repo? If you're running your own private spec repo, you probably don't care because the `pod spec push` command first updates your local clone of the spec repo before pushing up the commits.

Where this matters is when you're pushing to the public trunk. The best solution I've found (short of pushing your spec to the world before fully testing it) consists of simply creating your own sandbox spec repo.


