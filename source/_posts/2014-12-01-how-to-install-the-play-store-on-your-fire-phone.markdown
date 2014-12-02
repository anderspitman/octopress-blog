---
layout: post
title: "How to Install the Google Play Store on Your Amazon Fire Phone"
date: 2014-12-01 22:43:09 -0700
comments: true
categories: 
---

## Background 
The last few days Amazon has been
[having a "fire sale"](http://www.cnet.com/news/amazon-slashes-price-of-unlocked-fire-phone-to-199/)
selling their fire phone for 199USD unlocked and off contract, plus a year
of Amazon prime (~99USD). Given the hardware this was a little to good for me
to pass up. Arguably the biggest problem with the Fire Phone (and all Amazon's
devices) is that it doesn't have access to Google's Play Store, and the OS and
bootloader are locked down tight which makes it somewhere between difficult
and impossible to install ROMs at the moment.

One short term solution is to sideload the Google Play Store in order to install some of the missing apps. I spent a solid couple of hours trying to figure this out so thought I would summarize what I've learned. I can't take credit for this information. It's basically a combination of
[this XDA forum thread](http://forum.xda-developers.com/fire-phone/general/fire-phone-firesale-t2860852) and
[this blog post](http://www.epubor.com/how-to-install-google-play-on-kindle-fire.html). Those guys are the real wizards. I'm repeating the information here to make it easier for people with the Fire Phone to find. I can confirm that these steps work on the Fire Phone running FireOS 3.6.8.

## Steps
### Download the files
Download each of the following APK files:

1.  [Google Service Framework](https://www.dropbox.com/s/nlayv7mxwfa3wnu/GoogleServicesFramework.apk)
2.  [Google Login Service](https://www.dropbox.com/s/vlksi5ltazofw1q/GoogleLoginService.apk)
3.  [Google Play Services](https://www.dropbox.com/s/mmfh4dpvaqhr7uv/Google%20Play%20services_3.2.25%20%28761454-36%29.apk)
4.  [Google Play Store](https://www.dropbox.com/s/8uwbw6bcfiftb4m/Google%20Play%20Store%204.1.10.apk)

### Tansfer APKs to Phone
I swear this was the hardest part. You need to find a way to get the files onto
the phone. If you can plug the phone in and transfer over USB that would
probably be the easiest. I'm using Linux and didn't want to go throught the
trouble of figuring that out. I ended up
[sideloading Dropbox](https://www.dropbox.com/android) and transferring them
that way.

### Enable APK App Installation
On the phone, go into **Settings** > **Applications & Parental Controls** > **Prevent 
non-Amazon app installation** and flip the **App Installation** Switch to **ON**.

### Install File Explorer
Open the Amazon App Store and install ES File Explorer or another file browsing
app.


### Install the APKs
Using the file explorer (or Dropbox, etc), navigate to the files you
downloaded, and install them one by one in the same order you downloaded them. Be sure to
**reboot between each installation**. I don't know how important that is but
that's what I did and it worked. After they are all installed you should be
able to launch the Play Store, log in with your Google account, and start
installing stuff. Currently I'm using Hangouts and Gmail and they seem to work
fine. Maps basically works but has some corrupted visuals. YMMV.

NOTE: Initially I wasn't able to log into my google account because I had
Google's [2 factor authentication](https://www.google.com/landing/2step/)
enabled. I disabled it and it worked fine. If anyone finds a workaround let me
know.

Enjoy!

