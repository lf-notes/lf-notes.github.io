---
layout: default
title: Stream live WebM video to browser using Node.js and GStreamer
tags: webm http live stream nodejs gstreamer
comments: true
---
# Stream live WebM video to browser using Node.js and GStreamer

In this post, we'll stream live WebM video to the browser using just GStreamer and Node.js. We'll use Node.js with the Express middleware. We have used the latter to [stream a WebM file](_posts/2011/2011-04-25-stream-webm-file-to-chrome-using-node.js.md) on demand.

## Code

We receive a request from the browser at port 9001, create a TCP server socket to receive the WebM stream, and stream all data received from that socket to the browser. Then, we spawn a GStreamer pipeline to mux a WebM stream, and stream it to the TCP server socket using the `tcpclientsink` element.

The code follows

```javascript
var express = require('express')
var http = require('http')
var net = require('net');
var child = require('child_process');

var app = express();
var httpServer = http.createServer(app);

app.get('/', function (req, res) {
    var date = new Date();

    res.writeHead(200, {
        'Date': date.toUTCString(),
        'Connection': 'close',
        'Cache-Control': 'private',
        'Content-Type': 'video/webm',
        'Server': 'CustomStreamer/0.0.1',
    });

    var tcpServer = net.createServer(function (socket) {
        socket.on('data', function (data) {
            res.write(data);
        });
        socket.on('close', function (had_error) {
            res.end();
        });
    });

    tcpServer.maxConnections = 1;

    tcpServer.listen(function () {
        var cmd = 'gst-launch-1.0';
        var args =
            ['autovideosrc',
                '!', 'video/x-raw,framerate=30/1,width=320,height=240',
                '!', 'videoconvert',
                '!', 'queue', 'leaky=1',
                '!', 'vp8enc',
                '!', 'queue', 'leaky=1',
                '!', 'm.', 'autoaudiosrc',
                '!', 'queue', 'leaky=1',
                '!', 'audioconvert',
                '!', 'vorbisenc',
                '!', 'queue', 'leaky=1',
                '!', 'm.', 'webmmux', 'name=m', 'streamable=true',
                '!', 'queue', 'leaky=1',
                '!', 'tcpclientsink', 'host=localhost',
                'port=' + tcpServer.address().port];

        var gstMuxer = child.spawn(cmd, args);

        gstMuxer.stderr.on('data', onSpawnError);
        gstMuxer.on('exit', onSpawnExit);

        res.connection.on('close', function () {
            gstMuxer.kill();
        });
    });
});

httpServer.listen(9001);

function onSpawnError(data) {
    console.log(data.toString());
}

function onSpawnExit(code) {
    if (code != null) {
        console.log('GStreamer error, exit code ' + code);
    }
}

process.on('uncaughtException', function (err) {
    console.log(err);
});
```

## Execute

Assuming you have saved the code to a file called script.js, run Node.js thus

```bash
node script.js
```

Now, you can play the WebM stream in Chrome by accessing http://localhost:9001/.

## Debug

You may want to trace all system calls, especially if you change the args to GStreamer and get a cryptic message such as

```text
execvp(): No such file or directory
```

You can execute Node.js with strace

```bash
strace -fF -o strace.log node script.js
```

## Video and audio source elements

Here's a list of alternative video source elements

* autovideosrc - automatically detects and chooses a video source
* ksvideosrc - video capture from cameras on Windows
* v4l2src - obtains video stream from a Video 4 Linux 2 device, such as a webcam
* ximagesrc - video stream is produced from screenshots

Here's a list of alternative audio source elements

* autoaudiosrc - automatically detects and chooses an audio source
* alsasrc - captures audio stream from a specific device using alsa
* pulsesrc - captures audio stream from the default mic, based on system settings

An important point to note is that all these sources are live sources. GStreamer defines live sources as sources that discard data when paused, and produce data at a fixed rate thus providing a clock to publish this rate.

## Limitations

GStreamer pipelines cannot simultaneously capture streams using sources that access the same device, hence `tcpServer.maxConnections` has been restricted to 1. Even assuming that simultaneous access to the device was possible, the code above is CPU intensive since audio and video encoding is done once per viewer.
