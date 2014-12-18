---
layout: post
title: "Fix for Vimium that Stopped Working"
date: 2014-12-17 17:30:55 -0700
comments: true
categories: 
---

I love the 
[Vimium extension for Chrome](http://vimium.github.io/).
It basically provides VIM
keybindings for Chrome. But some of the bindings randomly quit working a
while back, probably after a Chrome update. A quick search didn't yield a
simple fix, so I just put up with it for an embarrassingly long time.
Finally today I did a bit more digging. Some of the issues on github seemed
to indicate local Chrome data might get messed up from updates. My solution
was to delete ```~/.config/google-chrome``` (actually
moved it to ```~/.config/google-chrome.bak``` just in case). I believe this
basically
removes all the local data for chrome, as if you had just installed it for the
first time. After starting Chrome back up and logging into my account, Vimium
is working again! I'm running Ubuntu 14.04 with Chrome 38 as of this
writing.
