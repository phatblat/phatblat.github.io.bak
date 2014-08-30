---
layout: post
title: "Configuring a KST Particle: iBeacon from 360|iDev"
date: 2014-08-29 21:43:46 -0600
comments: true
published: true
categories: 360idev ibeacon
---

{% img right /images/particle_back.jpg 150 'Particle iBeacon' 'an image of a Particle device showing a number on the back' %}

Everyone who attended this year's [360|iDev](http://360idev.com) in Denver was given a [Particle](https://kstechnologies.com/particle/) from KS Technologies. These are simple looking iBeacon devices that are fully configurable via KST's [Particle Accelerator](https://itunes.apple.com/us/app/particle-accelerator/id727105504?mt=8) app. KST also posted about this [awesome giveaway](https://kstechnologies.com/free-360idev-ibeacon/) on their blog.

### tl;dr

> The password scheme for the Particles given out at 360|iDev is: **360iDev123** — where _123_ is the number on the back.

<!-- more -->

# Battery Drain

First off, whenever you're not using your Particle, I suggest pulling the battery out or inserting something to prevent the flow of current. There's no "off" switch and the device will continually broadcast its iBeacon signal dilligently every 100 milliseconds (default rate) until the battery runs out of juice (40 days, according to the [Particle Calculator](https://itunes.apple.com/us/app/particle-calculator/id909745776?mt=8) app). The CR2032 coin cell batteries are cheap and easy to find, but you might as well save yourself the trouble and prevent the battery from draining when you're not using the device.

## Battery Removal

Taking out the battery is easy. The case is best opened with a Chuck E. Cheese token, but in a pinch a quarter, lightning connector, flat-head screwdriver or even a keyring works too. Just insert in the slot on the top and twist until the shell pops. The plastic shell is easily scratched, but that only affects your opinion of the device - not the performance.

After you pop the top off, use a pen to push the coin battery down toward the less-round end.

When you put the battery back in, the positive (+) side goes up.

{% img /images/particle_opening_slot.png 200 'Particle shell opening slot' 'an image of opening the Particle using a coin' %}
{% img /images/particle_battery.jpg 200 'Particle battery removal' 'an image of the Particle battery being removed' %}
{% img /images/particle_open.jpg 200 'Particle open with battery out' 'an image of the Particle internals' %}

# Config Prep

Before continuing, you'll need the following:

* iOS device with 7.0+ and BLE
  * YES: iPhone 4s/5/5s/5c, iPad 3/4/mini/Air, iPod "tall" (touch 5th gen)
  * NO: iPhone 4, iPad 2, iPod touch 4th gen
* the [Particle Accelerator](https://itunes.apple.com/us/app/particle-accelerator/id727105504?mt=8) app
* Bluetooth enabled

# Configuration

{% img right /images/particle_accelerator_app.png 200 'Particle Accelerator app' 'an image of the Particle Accelerator app showing my device' %}

Launch the Particle Accelerator app and tap the toggle switch in the upper right corner to enable scanning. Once your Particle appears, tap on it to bring up the configuration screen.

## Password

The password scheme for the Particles given out at 360|iDev is: **360iDev123** — where _123_ is the number on the back.

The number on the back of my Particle is 57, so the initial password to configure it was 360iDev57 (no leading zeros).

You can change the password once the current password is accepted. A blank password is allowed and does make it easier to use the app since you will be asked for the password each time you connect to the Particle. However, if you set a blank password, remember to either set a real password when your done or remove the battery. Anyone else with the Particle Accelerator app could muck with your new Particle if you forget.

## UUID

> NOTE: As of version 2.1.1, the Set UUID screen _always_ sets the UUID when you tap the "Done" button; there is no way to cancel, besides killing the app. So, only tap on the UUID on the configuration screen if you intend to change it.

The UUID essentially defines a group of iBeacons. In the CoreLocation API, each `CLBeaconRegion` is identified by the UUID, and there can be many iBeacons in each region. It is also possible to monitor for multiple beacon regions at the same time.

The Particles given out at 360|iDev all have their UUID set to **7D65B622-4AA8-4560-914C-502BE940BC16**. You can change this if you want. One of the UUIDs used by the AirLocate app (see below) is already in the list (E2C56DB5-DFFB-48D2-B060-D0F5A71096E0), so you can simply select this if you're wanting to test.

You can also give your Particle a unique UUID. On your mac, the `uuidgen` terminal command will generate a new UUID.

```
$ uuidgen                                                                                            
30FDAAFB-C585-4DA5-81F9-2CA293EFCF93
```

## Major & Minor Numbers

Within a single `CLBeaconRegion`, individual `CLBeacon` instances are identified by unique combinations of the `major` and `minor` numbers. The Particles from 360|iDev all have a `major` value of 1 and unique `minor` numbers that correspond to the number printed on the back of the shell.

# Testing your Particle

{% img right /images/particle_detector_app.png 170 'Particle Detector app' 'an image of the Particle Detector app showing my device' %}

## Particle Detector

https://itunes.apple.com/us/app/particle-detector/id724226138?mt=8

This free app from KST shows you details about any Particles that are nearby. It can only monitor one region UUID at a time, so it's easiest if you copy the UUID string from your browser and paste it into the filter screen. Then, make sure to select the UUID.

## AirLocate

https://developer.apple.com/LIBRARY/ios/samplecode/AirLocate/Introduction/Intro.html

This sample app was released at WWDC 2013 for testing iBeacons.

{% img right /images/roygbeacon.png 170 'RoyGBeacon app' 'an image of Roy G. Beacon app with a red background showing my device' %}

## RoyGBeacon

https://github.com/phatblat/RoyGBeacon

I created a simple app which changes the background color of the screen from gray to red depending on how close you are to an iBeacon. This app will work with any brand of iBeacon, but you may need to [modify the source](https://github.com/phatblat/RoyGBeacon/blob/master/RoyGBeacon/RGBMainViewController.m#L41) to add the UUID of your beacon (the AirLocate and 360|iDev UUIDs are included).

## BTLExplorer

https://itunes.apple.com/us/app/btlexplorer/id532751145?mt=8

Another app from KST. This one is a general discovery app for any Bluetooth device. Useful for monitoring signal strength and determining whether there is a problem at the hardware level (usually battery).
