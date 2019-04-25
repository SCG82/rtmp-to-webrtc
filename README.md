# rtmp-to-webrtc

Low latency (within 500ms) live broadcast system based on RTMP-CDN and WebRTC


### Demonstration

https://rtmp-to-webrtc.dot.cc

The demo is deployed on a personal test server with limited bandwidth. Please notify me if it hangs.


### How it works

-  RTMP is pushed to the CDN, and encoding parameters and gop parameter tuning are required.
-  Edge node deploys webrtc server
-  When the user accesses the video stream, the edge node webrtc server goes to the CDN to pull the stream.
-  Encapsulate rtmp into rtp, fed to webrtc server



### RTMP push script

Pushing part using ffmpeg
```
ffmpeg -f lavfi -re -i color=black:s=640x480:r=15 -filter:v "drawtext=text='%{localtime\:%T}':fontcolor=white:fontsize=80:x=20:y=20" \
-vcodec libx264 -tune zerolatency -preset ultrafast  -bsf:v h264_mp4toannexb  -g 15 -keyint_min 15 -profile:v baseline -level 3.0   \
-pix_fmt yuv420p -r 15 -f flv rtmp://39.106.248.166/live/live

```



### RTMP to package RTP

This part uses gstreamer, only the use of gstreamer is because the RTP package of ffmpeg is found, there is a certain probability that WebRTC will parse the failure, and no specific reason has been found.
```
gst-launch-1.0 -v  rtmpsrc location=rtmp://localhost/live/{stream} ! flvdemux ! h264parse ! \
rtph264pay config-interval=-1 pt={pt} !  udpsink host=127.0.0.1 port={port}

```


### some data

The server is deployed on Alibaba Cloud, and the delay is within 1000 milliseconds. The gstreamer's transcapsulation introduces a 300ms-500ms delay (visual inspection, not verified yet).
The overall delay after optimization can be within 500ms.







