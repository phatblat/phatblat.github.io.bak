---
layout: post
title: "Communicating Between an iOS App and a Watch Extension"
date: 2015-02-04 22:13:24 -0700
comments: true
categories:  ios watchkit architecture
---

I'm building a WatchKit extension for an existing iOS app and have been experimenting with the "plumbing" between the two. Watch extensions are different animals than other iOS 8 extensions, but there is a lot of overlap in this area. Today I made a diagram to show the options of getting data to the iPhone app from the Watch and vice-versa.

{% img /images/apple_watch_communication.png 'Communicating between an iOS App and a Watch Extension' 'a flowchart diagram' %}

This is simply a graphical representation of the summary presented in a [Developer Forums thread](https://devforums.apple.com/thread/256667?tstart=0).

The real gem in this is that Darwin notifications (`CFNotificationCenterGetDarwinNotifyCenter`) can be used to notify the other processes that there is new data available in the shared container. I haven't worked with Darwin notifications before, but the big difference from NSNotifications is that you can't pass any data along with it, just an identifier.

Note that "write data to shared container" includes any API that writes out files - just point it inside the NSURL returned `-[NSFileManager containerURLForSecurityApplicationGroupIdentifier:]`. 

## NSUserDefaults

If you're using NSUserDefaults, it's just an API on top of a plist file. When you stand up an NSUserDefaults instance using `-initWithSuiteName:` the data ends up in a file `<SHARED_CONTAINER>/Library/Preferences/<APP_ID>.plist` (where SHARED_CONTAINER is something like `~/Library/Developer/CoreSimulator/Devices/1A8DF360-B0A6-4815-95F3-68A6AB0BCC78/data/Container/Data/Application/`). That file will be written to every time you call `+synchronize` or at "periodic intervals".

## Other Tidbits

[MMWormhole](https://github.com/mutualmobile/MMWormhole) is a very neat library which handles a lot of these details for you providing a simple API for passing messages between an iOS app and extensions.

Tom Harrington has some great (extension development tips)[http://atomicbird.com/blog/ios-app-extension-tips].

Remember to [stay away from the file coordination APIs](https://developer.apple.com/library/ios/technotes/tn2408/_index.html).

