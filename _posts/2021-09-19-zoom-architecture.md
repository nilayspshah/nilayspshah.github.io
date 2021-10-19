---
layout: post
title: How Does Zoom Work?
subtitle: Understanding Zoom Application
tags: [architecture] 
comments: true
---

Have you ever wondered?
- How did Zoom scale(mostly) well with a sudden surge in users almost over-night?
- Why do they provide free usage for 2 person calling, but 40min limit for multi person call? 
- Why does it want us to install its client and not do it over browser only?

The answer lies in zoom's architecture

### First stab at Zoom Flow
Intuitively, this is what the flow for a video conferencing application seems to be
- Get individual data(video/audio) from all user streamings
- Transmit Data to server
- Merge the stream of data of all users in the call
- Send most optimised data stream to all users, based on their network connectivity etc.

### High Level Components
- Zoom Client
  - Web
  - Native
- Zoom Servers 
    - Multimedia Routers
    - Zone Controllers
    - Global Cloud Controller

### What seems to be Zoom's "secret" sauce?
It is very important to understand which part of the flow is done by which component.
- The first intuition we had was problematic, because it is very difficult to scale.
- Essentially, as per original idea, we send the traffic to a datacenter, transcode it into a single view common for everyone, and then send mixed video out to every individual participant. That introduces latency, uses a lot of CPU resources, and it's hard to scale and deploy new datacenters to meet increased load.

#### Client -> Server : Sending your data

- Zoom does Video encoding/decoding on the client devices, saving on important computing resources.
Encoding is required to reduce redundancy in data being transmitted, over low internet bandwidth
- (Going on a bit low level detail now but it is interesting) Zoom uses SVC(Scalable Video Codec) technique, alternative being AVC(Advanced Video Codec) technique
- SVC can send data over single stream at multiple bitrates, AVC requires multiple streams for multiple bitrates
- For Eg. If user is on a bad network and he could send low quality video on a stream, when their internet improves, they can send the improved video over the same stream, AVC would have required different streams for different quality videos
- SVC creates a base layer with lowest bitrate, and builds upon it with higher bitrates, pretty nifty stuff!
  
  ![Crepe](https://pbs.twimg.com/media/E94siu_VkAUteGC?format=png&name=medium)

#### Processing on server
- As we mentioned that zoom doesnt transcode anything on its server, you as a client get streams of data from individual members in the call
  - Multimedia router manages these streams that will be sent to you.
- Now coming to transportation, Zoom uses mixture of TCP/UDP/TLS/HTTPS protocols
- Zoom has distributed data centers across multiple locations, your client will connect to nearest data center.
- In 2015, Zoom partnered with Equinix and had 13 data centers all over the world. Since then,  Equinix helps with the key business, specifically the financial services and government sectors. Zoom has 17 data centers in 4/2020.
- Nearby users get assigned to same data center, to reduce network latency(150 ms for network roundtrip from CA->Netherlands->CA) / or if not nearby, they get assigned to least loaded server
 
#### Zoom QoS(Quality of Service)
- Zoom has QoS at application layer, telemetry data is collected to determine the network conditions of the client. this is communicated to the server
- Data QoS looks at includes, CPU, network jitters, packet loss etc
- If you have a bad network, then 
  - send low quality data back so that your downstream bandwidth is utilised
  - subscribe to low quality data(possible over the same stream thanks to SVC) of other participants, this MMR manages

### Unlimited Free Calling for 2 people but 40 minute Limit for more
- In case there are just 2 participants, peer to peer connection is used by zoom. This one of the reasons why, 2 person meeting is free, minimal compute cost.
- In case of more than 2 people, the call needs to be "managed", hence it incurs cost to company as their server hardware is utilised

### Unsubtle Nudge to use client instead of browser
- Zoom uses websockets to manage their connections via browser, which uses TCP, which leads to bad performance in case poor networks. UDP connections are not supported directly via websockets,
- For that zoom would need to use WebRTC, which they started using but in a different way..
- Essentially, to avoid browser as an intermediary, zoom nudges us to use their client as it gets more control over the experience 


Sources:

 - https://lavivienpost.com/how-zoom-works/
- https://sjsu.edu/workanywhere/docs/Zoom%20Message.pdf
- https://webrtchacks.com/zoom-avoids-using-webrtc/
- https://bloggeek.me/when-will-zoom-use-webrtc/
- https://blog.zoom.us/zoom-can-provide-increase-industry-leading-video-capacity/
- https://medium.com/@vsachdeva/zoom-video-conf-tool-at-scale-e86289c290b8
- https://web.wpi.edu/Pubs/E-project/Available/E-project-031020-183422/unrestricted/ebactsIQPreport.pdf
- http://highscalability.com/blog/2020/5/14/a-short-on-how-zoom-works.html