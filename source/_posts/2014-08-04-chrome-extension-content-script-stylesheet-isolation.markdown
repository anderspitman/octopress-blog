---
layout: post
title: "Chrome Extension Content Script Stylesheet Isolation"
date: 2014-08-04 08:52:47 -0700
comments: true
categories: [Chrome Extension, HTML, HTML5, CSS, CSS3, JavaScript, Tutorial]
---

## Background Info

When writing Chrome extensions, if you want to inject HTML and CSS into pages the user is
visiting, you use what's called a
[content script](https://developer.chrome.com/extensions/content_scripts).
One reason you might want to do this would be to build a custom popup
that activates on certain pages.

One of the biggest problems people run in to is CSS corruption. The way that content
scripts work means that the CSS from your content script is merged with the CSS
from the page the user is visiting. This means that the page can corrupt what
your popup looks like, and the popup might mess up the page.
[See here](http://stackoverflow.com/q/12783217/943814).
The ideal
situation is for your content script to run in a completely isolated environment.
Unfortunately this isn't straightfoward. There are 
[a couple different options](http://stackoverflow.com/a/20241247/943814). The choice came down to IFrames vs Shadow DOM. I decided to try
Shadow DOM first.

The Shadow DOM is (as of this writing) a new technology that is part of the upcoming
[Web Components](http://webcomponents.org/). It's very cool stuff.
When first trying to implement my popup I tried using the Shadow DOM, but I ran into 
problems when 
[trying to run JavaScript](http://stackoverflow.com/q/25048359/943814)
in my popup. This
[led me to Custom Elements](http://stackoverflow.com/a/25053376/943814),
another web components feature. Since both shadow DOM and custom elements are very new
and not universally supported, at this point I decided to try 
[Polymer](http://www.polymer-project.org/).
Polymer is a project that provides nice wrappers around web components features, as well
as 
[polyfills](http://remysharp.com/2010/10/08/what-is-a-polyfill/) for features that aren't
implemented natively yet. Polymer turned out to be awesome, and did exactly what
I need, but unfortunately there is 
[a bug](https://code.google.com/p/chromium/issues/detail?id=390807)
in the current version of chrome that
prevents custom elements from working in content scripts. Back to square one.

Alright, that leaves us with the infamous iframe. This is the solution that
worked for me. In the end it was pretty strightforward. There are a couple
caveats, but nothing too bad. I'll run through the basics of how I implemented
it.

All of the code used in this example is available from the following github repo:
https://github.com/andersp/chrome-extension-css-isolation-example

## Create a new Chrome Extension

Create an empty directory and add the following manifest.json:

```json manifest.json
{
  "manifest_version": 2,
  "name": "CSS Isolation",
  "description": "This extension demonstrates Content Script CSS Isolation",
  "version": "1.0",
  "background" : {
    "scripts" : ["background.js"]
  },
  "content_scripts" : [
    {
      "matches" : ["<all_urls>", "http://*/*", "https://*/*"],
      "css" : ["content.css"],
      "js" : ["content.js"]
    }
  ],
  "web_accessible_resources" : ["popup.html"]
}
```

## Add Content Script and Style

The manifest references several files that we will need to create. Let's start
with content.js:

```javascript content.js
var iframe = document.createElement('iframe');
iframe.src = chrome.extension.getURL("popup.html");
iframe.className = 'css-isolation-popup';
iframe.frameBorder = 0;
document.body.appendChild(iframe);

chrome.runtime.onMessage.addListener(function(message) {
  iframe.style.display = 'none';
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

## This is Important

One other thing you'll notice from content.js is the chrome message handler.
This brings up a very important point and huge caveat of content scripts in
general, and especially using iframes within content scripts. You cannot
directly access code within an iframe from other parts of your extension.
It must use the chrome message
passing to transfer information. In addition to this, the iframe
cannot pass messages directly to the content script. Therefore, the
iframe and content script must communicate with each other through
the background page. This is explained in more detail
[in this excellent post](http://www.sitepoint.com/chrome-extensions-bridging-the-gap-between-layers/).
I think this will be much more clear once we finish our example.

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
<script src='popup.js'></script>
</body>

</html>
```

Mostly just styling. The overlay is a shaded region which will fill the window
surrounding our small popup. The popup lives inside the wrapper. I want to stress the
fact that everything in here is completely isolated from whatever page the user
is visiting. We can name our classes whatever we want with no fear of
name collisions from the outside world. Perfect!

We're sourcing popup.js from within popup.html. There's no need to
add it in the manifest.

```javascript popup.js
chrome.runtime.onMessage.addListener(function(message) {
  if (message == 'hide_popup') {
    iframe.style.display = 'none';
  }
});
```

Here we're handling when the user clicks outside the popup, in the overlay
region. When this happens we want to signal the content script to hide
our iframe. But remember what we said earlier: we can't communicate
directly with the content script, so we need to send the message to
the background page and have it forward it to the content script.

## Add Background Page

Add the background page as follows:

```javascript background.js
chrome.runtime.onMessage.addListener(function(message, sender) {
  chrome.tabs.sendMessage(sender.tab.id, message);
});
```

Literally all it does is repeat whatever messages it receives back out to the
tab it received it from. It's worth noting here that both content.js and 
popup.js will receive the forwarded message, so it's actually being
reflected back to the popup where it originated.

So at the end of the day, here's what happens:

1. User clicks shaded region
2. popup.js detects the click and sends the message `hide_popup` to background.js
3. background.js receives the message, and broadcasts it to the tab where it originated
4. content.js receives the message, and if it is `hide_popup` it hides the iframe

## Conclusion

And there you have it! Load this puppy into chrome, and any page you visit should
display a popup. Clicking in the faded area around it makes it disappear.
This is a barebones example to be sure but it should
be fairly straightforward to augment with additional functionality.

