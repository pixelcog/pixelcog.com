---
layout:  post
title:   Disable .DS_Store in OS X El Capitan
tags:    osx asepsis ds_store el capitan
image:   asepsis-el-capitan.jpg
---

Like many OS X users, I *hate* .DS_Store files cluttering up my folders.  For the average OS X user working in the Finder these are hidden safely away from view, but for most programmers and really anyone who works in the Terminal these little garbage files tend to get in the way.

{{ more }}

## Asepsis to the Rescue!

For several years, Asepsis has been one of the first tools I would install on a fresh OS X installation to block the writing of .DS_Store to all of my directories as I browse around in the Finder.  The way Asepsis works is by installing a wrapper around a specific OS X system framework which intercepts all attempts to create or write to a .DS_Store file and it redirects them to a special location within `/usr/local/.dscage`.  This way Finder still operates as normal without polluting the file system with this nonsense.

Unfortunately, with the release of OS X 10.11 El Capitan, Apple has enabled *[System Integrity Protection (SIP)](https://en.wikipedia.org/wiki/System_Integrity_Protection)* which prevents Asepsis from installing and functioning as normal.  The lead developer has dropped all support for the utility citing (understandably) that he doesn't want to direct users to disable critical security services in order to use this tool.

### Running Asepsis in El Capitan Without Compromising Security

Well it turns out that we can still get this to work, **AND** keep SIP enabled.  Win-win, right?  How does this work?

The following are the steps I've taken to get this working again:

#### 1) Install Asepsis

First, open up a terminal and run the following commands:

    touch ~/.no-asepsis-os-restriction
    touch ~/.asepsis-suppress-update-errors

This will bypass Asepsis' built in compatibility check (since it doesn't officially recognize El Capitan).

Now you can install the latest version of Asepsis (1.5.2 as of Feb 2016) from [the official website](http://asepsis.binaryage.com) ([direct link](http://downloads.binaryage.com/Asepsis-1.5.2.dmg)).  Once you have run the installer, it will make you reboot.  Unfortunately at this point, the tool will be prevented from running due to SIP.

#### 2) Enter recovery mode and disable SIP

Reboot your computer and hold down Command (&#8984;) + R keys simultaneously once you hear the startup chime.  Your computer will now boot into "recovery mode".

Once you see the "OS X Utilities" screen, go to the "Utilities" drop down menu at the top of the screen and select Terminal.  Once you have a command prompt, type:

    csrutil disable; reboot

Your computer should now reboot itself.

#### 3) Install Asepsis with SIP disabled

Once you're back in non-recovery mode, open a terminal and run the following command to install Asepsis

    asepsisctl install

> **Note**: If you get a warning about the wrapper already existing (perhaps from a previous installation attempt), try running `asepsisctl uninstall_wrapper` and then re-running `asepsisctl install`.

#### 4) Re-enable SIP

Once again reboot your computer and hold down Command (&#8984;) + R keys to enter Recovery Mode, then open up the Terminal from the Utilities menu.  Re-enable SIP with the following command:

    csrutil enable; reboot

#### 5) Verify Asepsis is running

After your computer has finished rebooting yet again, open up a Terminal and type:

    asepsisctl diagnose

If all went well you should see: **Your Asepsis installation seems to be OK**.

Congratulations, you now have Asepsis running *with* SIP still turned on.  If you go ahead and create a new folder, open it up in Finder, add some files, and mess with the view options, then check the contents, you'll notice a conspicuous absence of `.DS_Store` shenanigans.
