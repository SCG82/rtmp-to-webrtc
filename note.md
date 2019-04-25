** Develop large-scale low-latency (within 1000 milliseconds) live broadcast system based on RTMP and WebRTC

**** Question

With the large-scale popularization of mobile devices and the increasing cost of traffic, there are more and more low-latency scenes. From last year to this year, there are online doll machines, live answering questions, online karaoke, etc. To achieve ultra-low latency of audio and video is not easy, encoding delay, network packet loss, network jitter, multi-node relay, video segmentation transmission, playback side cache, etc. will bring delay.

**** WebRTC's rise to the solution and problems encountered

The rise of WebRTC technology has brought solutions for low-latency audio and video transmission, but WebRTC is designed for end-to-end, suitable for real-time interactions on a small scale, such as video conferencing, even wheat scenes. Even with the addition of SFU Media Server as a forwarding server, it is also difficult to achieve large-scale distribution. Another need to consider the cost of traffic, WebRTC real-time traffic is transmitted via UDP (in some cases can use TCP), can not be reused in traditional CDN On the architecture, the real-time traffic price is more than three times that of CDN traffic. The cost of deploying an ultra-low latency live broadcast network is very high.

**** RTMP system push stream playback delay analysis

An optimized RTMP-CDN network has an end-to-end delay of approximately 1-3 seconds, and a delay of more than 5 seconds or more. From push to playback, delays are introduced with coding delays, network packet loss and Network jitter, video segmentation, multimedia node relay, player cache, etc. In fact, in addition to network packet loss and network jitter is not controllable, other aspects have certain optimization options, such as Using x264's -preset ultrafast and zerolatency, the encoding delay can be reduced. The segmented transmission part can reduce the GOP to less than 1 second. The buffer can be appropriately reduced on the player side, and a certain chasing frame strategy can be set to prevent excessive The delay caused by the buffer.

**** Low cost low latency implementation

In the RTMP live broadcast system, deep customization from the push-stream to the network transmission to the player can achieve relatively low latency, but the cost is relatively high, requiring a complete high-level team (server and client). And a lot of bandwidth server resources. If you want to achieve ultra-low latency (less than 1000 milliseconds) is even more difficult, and such a low delay will bring some negative effects, the network will appear when there is a little jitter Don't wait. Is there a lower cost implementation? And how to reuse the existing CDN infrastructure to achieve low latency? In fact, we can make some optimization adjustments on the existing RTMP-CDN system. The node converts the RTMP stream into a stream that WebRTC can play to achieve low latency and multiplexing of the CDN system. At the same time, WebRTC anti-lost packet can be used to optimize the last mile viewing experience. WebRTC has a corresponding SDK on each platform. Especially in the browser embedded, can greatly reduce the development of the entire system, upgrade, maintenance costs, to achieve the effect of opening the browser.

**** Questions to be aware of

Of course, things can't be perfect. The world is a bug. Let RTMP and WebRTC interoperate well and do some extra work:

1, RTMP push-end low latency and GOP size

If you want to achieve low latency, we need to be as fast as possible on the push stream. At the same time, RTMP-CDN will generally have a GOP cache, which will cache the latest GOP. If the GOP is too large, it can't be low-latency. The GOP is set at 1 second. One of the benefits is that on the WebRTC player, if there is a key frame loss, you can reply quickly. In our scenario, the WebRTC server will reject the FIRRTR FIR message, through the next keyframe. Solve the problem of missing key frames.

2, RTMP source station and edge station do not do any cache as much as possible

In a live stream with a frame rate of 25 FPS, a frame buffer will increase the delay by 40 ms. In our scenario, the source and edge stations of the RTMP should be as small as possible except for some GOP caches.

3, encoder parameter settings

WebRTC's support for H264 is not perfect. For example, chrome supports H264's baseline, main profile and high profile. Firefox and safari currently support baseline. Although the existence of B-frames can reduce some bandwidth usage, it will introduce more delays. Recommended use. After testing the encoding parameters of H264, you can choose to be baseline level3.

4, PPS and SPS

In the RTMP scenario, we usually only add PPS and SPS at the beginning of the push, but WebRTC requires PPS and SPS in front of each keyframe. This problem can be solved when pushing the stream, or RTMP can be used. Join when converting to RTP. The versatile ffmpeg already supports this bitstream filter -- dump_extra, thank you ffmpeg for saving audio and video developers so much time.

5, audio transcoding

In RTMP protocol specification, audio supports pcma and pcmu. WebRTC also supports pcma and pcmu. If the audio and video pushed by RTMP push is in pcma or pcmu format, we don't need to transcode. Of course, the reality is cruel, big in RTMP system. Most manufacturers and open source projects only support AAC. At this time, we need to transcode the audio. This kind of work is only one or twenty lines of code for the versatile ffmpeg. Thanks again ffmpeg for the audio and video developers to save. More time. (See ffmpeg's powerful, if you want to learn ffmpeg, please buy the master's book <<FFmpeg from entry to master>>

6, video to package

In the video section, we mentioned that H264 baseline should be used as much as possible. In this case, WebRTC support will be better. We only need to encapsulate the RTMP stream into RTP stream and feed it to the corresponding WebRTC mediaserver. This part can be done with FF mpeg. .

**** How to land

At present, there is no need for complete matching at all. This solution does not currently fall to the ground. The proposed way of landing is that the RTMP part uses the existing CDN to deploy the edge node of WebRTC and pulls the CDN according to the access request. The only part of the WebRTC media server is the edge node. In this part, we can use some open source media server and then do some business development. The open source WebRTC mediaserver that supports rtp input has janus-gateway, medooze mediaserver.

Talk is cheap, show me the code. I implemented a prototype implementation of RTMP push-stream WebRTC playback, and the test delay on Alibaba Cloud is within 500ms. The complete code is here https://github.com/RTCEngine/rtmp-to -webrtc

**** At last

The last and final, of course, is the advertising process. I have joined the school and think about the online school, responsible for the research and development of interactive live broadcast products. Currently there are many pits in the audio and video direction, the client and the server are relatively lacking, if the audio and video and WebRTC And interested in online education, please contact me.

Please send me your resume: leeoxiang@gmail.com
