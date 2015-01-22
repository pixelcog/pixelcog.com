---
layout:  post
title:   Flat Byword Icon for Yosemite
tags:    byword yosemite icon flat design osx
image:   byword-icon.png
updated: 2015-01-21 @ 7:06pm
---

Since upgrading to Yosemite, I've been loving the new design and the flat icons.  However, the few old-school icons left on my dock are now a complete eye sore by comparison.  Since the Byword icon is staring back at me most of the work day I decided to go in update it.

{{ more }}

[♫♪ *one of these things is not like the other...* ♪♫][cookiemonster]
[![Byword icon in the dock](/img/posts/byword-icon-dock.jpg)][cookiemonster]

Before and after... My OCD brain is no longer screaming at me.

I really liked the way [Atom](http://atom.io/), and [Instacast](http://vemedio.com/products/instacast-mac) updated their icons for Yosemite (matching iTunes and other native apps), so I decided to emulate the same look with the white border and a slight gradient.

[cookiemonster]: https://www.youtube.com/watch?v=TxfZbj5vf7Y "one of these things is not like the other..."


## Updating the Icon

To use this icon, simply download it [here][icon-file], then find your Byword app, select *Get Info* (⌘I), and drag the Byword.icns file into the icon in the upper left corner of the info pane:

![Byword icon in the dock](/img/posts/byword-icon-drag.jpg)

> **Note:** a lot of icon tutorials tell you to go into the .app file's resources and replace the default icon within the application itself.  This **[will not work](http://forums.macrumors.com/showthread.php?t=1105471)** with Mac App Store apps like Byword because they sign their code and any change to the package's contents will require a new signature.  Using the "*Get Info*" method is the only way to do this.

You may need to `killall Dock` before the new icon will show up for you.

## Rounded Yosemite Icon Template

[![Yosemite icon template](/img/posts/byword-icon-template.jpg){: .pull-right-responsive}][psd-file]

If you want to create your own rounded Yosemite-style icon like this one, you can download the PSD file [here][psd-file].

The process is pretty simple.  Essentially you'll need to export versions of your icon as png images ranging from 16x16 @ 72ppi to 512x512 @ 144ppi (which is actually 1024x1024), then run a command to compile them all into an icon.  If you have Xcode installed on your system, it comes bundled with the `iconutil` command needed to turn your image into an *icns* file.

There's a nice tutorial at [davenelson.com](http://davenelson.com/make-icon-mac-yosemite/) explaining how to do all of this.

Enjoy!

[icon-file]: http://pixelcog.com.s3.amazonaws.com/blog/byword-icon.zip "Byword.icns"
[psd-file]: http://pixelcog.com.s3.amazonaws.com/blog/icon_template.psd.zip "icon_template.psd"
