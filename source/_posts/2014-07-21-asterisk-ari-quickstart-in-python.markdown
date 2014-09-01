---
layout: post
title: "Asterisk ARI Quickstart Tutorial in Python"
date: 2014-07-21 10:41:56 -0700
comments: true
categories: [Python, Asterisk, ARI, Quickstart, Tutorial]
---

The purpose of this post is to get Asterisk users up and running with the Asterisk 12 ARI
with Python as quickly as possible. I'm assuming:

* You know what the ARI is
* You know at least the basics of using Asterisk
* You have Asterisk 12 installed
* You have Python with pip installed (preferably inside a virtualenv)

I followed this other tutorial closely, particularly the implementation of
the websocket stuff:

https://wiki.asterisk.org/wiki/display/AST/Getting+Started+with+ARI

For more info refer to the [Official ARI Page](https://wiki.asterisk.org/wiki/display/AST/Asterisk+12+ARI)

Note that I'm implementing my own interface for the REST calls, since it's
a simple example. For a full blown application you'll probably want to use something
like [python-ari](https://github.com/kickstandproject/python-ari)


## Set up Asterisk 

### Enable HTTP server
Asterisk's HTTP server is disabled by default. Open **http.conf** and make sure
the following are uncommented.

```
enabled=yes
bindaddr=127.0.0.1
```

### Enable and set up ARI
Open **ari.conf** and uncomment:
```
enabled=yes
```

And add the following to the end of the file:

```
[hey]
type=user
password=peekaboo
```

### Create an extension
We need an entry point for Asterisk to pass control into our ARI app. Just
set up an extension that opens the Statis app as shown below. I'm using
extension 100 in the example **extensions.conf**:
```
[default]
exten => 100,1,Noop()
      same => n,Stasis(hello,world) ; hello is the name of the application
                                    ; world is its argument list
      same => n,Hangup()
```


## Get the Code

Either clone my repo at https://github.com/anderspitman/ari-quickstart or
just copy and paste the script from below.

You'll need to install requests and websocket-client. If you cloned the
repo just do:
```
pip install -r requirements.txt
```

Otherwise install them manually:
```
pip install requests websocket-client
```


``` python
#!/usr/bin/env python

import json
import sys
import websocket
import threading
import Queue
import requests


class ARIInterface(object):
    def __init__(self, server_addr, username, password):
        self._req_base = "http://%s:8088/ari/" % server_addr
        self._username = username
        self._password = password

    def answer_call(self, channel_id):
        req_str = self._req_base+"channels/%s/answer" % channel_id
        self._send_post_request(req_str)

    def play_sound(self, channel_id, sound_name):
        req_str = self._req_base+("channels/%s/play?media=sound:%s" % (channel_id, sound_name))
        self._send_post_request(req_str)

    def _send_post_request(self, req_str):
        r = requests.post(req_str, auth=(self._username, self._password))


class ARIApp(object):
    def __init__(self, server_addr):
        app_name = 'hello'
        username = 'hey'
        password = 'peekaboo'
        url = "ws://%s:8088/ari/events?app=%s&api_key=%s:%s" % (server_addr, app_name, username, password)
        ari = ARIInterface(server_addr, username, password)
        ws = websocket.create_connection(url)

        try:
            for event_str in iter(lambda: ws.recv(), None):
                event_json = json.loads(event_str)

                json.dump(event_json, sys.stdout, indent=2, sort_keys=True,
                          separators=(',', ': '))
                print("\n\nWebsocket event***************************************************\n")

                if event_json['type'] == 'StasisStart':
                    ari.answer_call(event_json['channel']['id'])
                    ari.play_sound(event_json['channel']['id'], 'tt-monkeys')
        except websocket.WebSocketConnectionClosedException:
            print("Websocket connection closed")
        except KeyboardInterrupt:
            print("Keyboard interrupt")
        finally:
            if ws:
                ws.close()


if __name__ == "__main__":
    app = ARIApp('localhost')
```

## Try it Out
Start/Restart Asterisk and once it's up run the script:
```bash
python ari-quickstart.py
```

If it doesn't throw any exceptions it should be connected and listening for ARI
events. Dial the Statis extension (100 in my case) and you should hear monkeys.

The script should be easy to modify to add more functionality. It's a good
starting point for creating more full featured apps. The biggest thing
to worry about is that there's a good chance you won't want your app
blocking on the websocket receive calls. A simple solution is to handle
events in a separate thread and use a Python queue to pass the received
messages in.
