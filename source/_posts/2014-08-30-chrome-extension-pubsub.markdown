---
layout: post
title: "Chrome Extension PubSub"
date: 2014-08-30 22:35:24 -0700
comments: true
categories: [Chrome Extension, HTML, HTML5, CSS, CSS3, JavaScript, Tutorial]
---

This tutorial builds the same Chrome extension popup as my
[Chrome Extension Content Script Stylesheet Isolation](http://anderspitman.com/blog/2014/08/04/chrome-extension-content-script-stylesheet-isolation/)
tutorial, but uses the [chromeps](https://github.com/andersp/chromeps) pubsub module to make things easier.
For more detailed information, I highly recommend looking through that tutorial.

You can get all the code for this tutorial from https://github.com/andersp/chrome-extension-pubsub-example

## Background Info
When writing chrome extensions with content scripts, you often find yourself doing a lot of message passing.
If your content scripts include iframes, things get even more complicated because in order to communicate 
between the content scripts and their iframes, you have to ferry the messages back and forth using the background
page. This can get messy very quickly. This tutorial serves as a simple but complete example of how to use
chromeps to help with these issues.

## Objective
To recap from the previous tutorial: we'll be creating a simple chrome extension that uses a content scripts with
a popup that loads on every page the user opens. When the user clicks outside the popup it disappears. This
demonstrates the different types of message passing mentioned above.

## Install chromeps
Create a new empty directory for you extension and download `chromeps.js` into it. You can get it from
https://github.com/andersp/chromeps

## Create a new Chrome Extension

Add the following manifest.json:

```json manifest.json
{
  "manifest_version": 2,
  "name": "Chrome Extension PubSub",
  "description": "This extension demonstrates Content Script CSS Isolation with chromeps",
  "version": "1.0",
  "background" : {
    "scripts" : ["chromeps.js"]
  },
  "content_scripts" : [
    {
      "matches" : ["<all_urls>", "http://*/*", "https://*/*"],
      "css" : ["content.css"],
      "js" : ["chromeps.js", "content.js"]
    }
  ],
  "web_accessible_resources" : ["popup.html"]
}
```
Notice that we are loading `chromeps.js` into the background page (for this example we actually don't have any
other logic for the background page), and also loading it each time a content script is loaded, which in this
case means any time the user opens a web page.

## Add Content Script and Style

The manifest references several files that we will need to create. Let's start
with content.js:

```javascript content.js
var iframe = document.createElement('iframe');
iframe.src = chrome.extension.getURL("popup.html");
iframe.className = 'css-isolation-popup';
iframe.frameBorder = 0;
document.body.appendChild(iframe);

chromeps.subscribe('commands', function(message) {
  if (message == 'hide_popup') {
    iframe.style.display = 'none';
  }
});
```

Here we're creating the iframe that will hold our popup. Try to make sure the
`className` is something unique because this is the one style that may
still interfere with the page the user visits. I'm using `css-isolation-popup`.
That style comes from content.css, which is referenced in the manifest. Let's
add it real quick:

```css content.css
.css-isolation-popup {
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;
}
```

I'm basically just giving the popup free reign over the entire window. It's fine in
my case because I have a shaded overlay that surrounds the actual popup. You might need
to tweak this for your needs.

Note that we've used chromeps to subscribe to the "commands" topic, so our callback will be invoked
any time a message on that topic is published anywhere in chrome.

## Add Popup

Now let's add the actual popup files, popup.html and popup.js:

```html popup.html
<!doctype html>
<html>

<head>
<style>
.overlay {
  position: fixed;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
  background-color: black;
  z-index: 1000;
  opacity: .80;
}
.wrapper {
  position: fixed;
  top: 50%;
  left: 50%;
  width: 400px;
  height: 200px;
  margin-left: -200px;
  margin-top: -100px;
  text-align: center;
  background-color:#FFFFFF;
  z-index: 1100;
}
</style>
</head>

<body>
<div class='overlay'></div>
<div class='wrapper'>
  <h1>Click outside to hide</h1>
</div>
<script src='chromeps.js'></script>
<script src='popup.js'></script>
</body>

</html>
```

Mostly just styling. The overlay is a shaded region which will fill the window
surrounding our small popup. The popup lives inside the wrapper. 

We're sourcing `popup.js` from within `popup.html`. There's no need to
add it in the manifest. We're also including `chromeps.js`.

```javascript popup.js
var overlay = document.querySelector('.overlay');
overlay.addEventListener('click', function() {
  chromeps.publish('commands', 'hide_popup');
});
```

Here we're handling when the user clicks outside the popup, in the overlay
region. When this happens we want to publish a signal to the content script to hide
our iframe.

## Conclusion
And that's it. If you compare this to the previous tutorial, you'll notice that we don't need to explicitly
create a background page just for passing messages, since chromeps takes care of all the heavy lifting for us.

