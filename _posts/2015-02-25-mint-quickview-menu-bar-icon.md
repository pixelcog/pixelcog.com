---
layout:  post
title:   Yosemite Menu Bar Icon for Mint QuickView
tags:    mint yosemite icon menubar flat design osx dark
image:   mint-quickview.png
---

Since upgrading to Yosemite, I've wanted to switch my interface to "dark mode", but there were a few incompatible menu bar apps holding me back.  One of these was [Mint QuickView](https://itunes.apple.com/us/app/mint-quickview/id533491939?mt=12), which both looked awful with a dark menu bar and contained an old, outdated logo. To remedy this, I created a quick drop-in replacement for the ugly icon.

{{ more }}

Annoyingly, Mint QuickView has not been updated since Feb 14th, 2013, almost two years ago as of this post.  Customer requests for a refresh have been met with notifications that Intuit has [no plans to update QuickView going forward](https://mint.lc.intuit.com/questions/1004048-yosemite-updates-coming). It's a shame because the app still functions well and I use it all the time.

Luckily, this app does not sign its resources, so we can go in and update the icon ourselves.  Taking Intuit's new Mint logo, I decided to go in and do just that. Here is a quick before and after comparison for the *colored* and *uncolored* icon in both dark and light menu bar modes:

![Mint QuickView Menubar before and After](/img/posts/mint-quickview-before-after.jpg)

## Installation

If you want to do this yourself, simply download and extract [mint-quickview-icon.zip](http://pixelcog.com.s3.amazonaws.com/blog/mint-quickview-icon.zip), then find your Mint QuickView.app in *Applications*, right click it and select "Show Package Contents":

![Show Package Contents](/img/posts/mint-quickview-package-contents.jpg)

Then, locate the "Resources" folder, and drag all of the extracted images directly into it:

![Drag Icon Resources](/img/posts/mint-quickview-resource-drag.jpg)

You will be prompted to enter your password, and then asked whether you wish to replace these files.  Simply check "Apply to All" and then select "Replace".

Re-launch the QuickView app and it should be sporting a new look that hopefully will no longer cause your eyes to bleed.