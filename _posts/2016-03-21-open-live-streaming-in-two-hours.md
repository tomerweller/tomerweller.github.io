---
layout: post
title:  "Setting up an open source live streaming solution in two hours"
description: Going through an extremely quick setup of a free video streaming solution
image: /assets/open-streaming/view-screenshot2.jpg
date:   2016-03-21 23:59:59 -0500
--- 

Last Sunday the Media Lab hosted a Public Dialogue on DRM and the future of the Web. Amongst the speakers was [Richard Stallman][stallman] (aka **RMS**), founder of the Free Software Foundation.  Although there was a lot of interest in the event from outside of the media lab, it couldn’t be streamed with MIT’s setup even though it was being filmed. MIT's video streaming service is proprietary and Stallman will not use, or even take part in using, software that is not *completely* free.
 
![The Panel](/assets/open-streaming/the-panel.jpg "this is a title")
<small>Richard Stallman, Danny O'Brien, Joi Ito, and Harry Halpin. Photo: Jon Christian</small>
 
Earlier that day, during a lunch hosted by [Joi Ito][joi], Tal Achituv and myself toyed with the idea of deploying a WebRTC based solution that is open enough for even RMS to approve. After doing some research, it seemed that there are indeed open WebRTC options for broadcasting. Now, a couple of hours before the panel, we decided to go for it. 

This resulted in two hours of hectic coding, setup and equipment search.

### The Setup: 

![The Setup](/assets/open-streaming/the-setup.jpg)
<small>Photo: Tal Achituv</small>

![Block Diagram](/assets/open-streaming/block-diagram.png)
<small>Making this diagram took more time than building a free streaming solution<small>

#### Media Server
[Kurento][kurento] (LGPL-2.1) is an open source media server. It implements the WebRTC spec and uses [GStreamer][gstreamer] under the hood for any multimedia processing. In this case we used Kurento as a broadcasting server: it received one WebRTC AV stream from a presenter and retransmitted it via multiple WebRTC streams to viewers. 

We ran Kurento on a Linux VM on my laptop. The intention was to test on a VM and then deploy to some cloud service. However, due to lack of time, the debug setup became the production one. 

Notes:

- After the Stream ended I realized that the VM had exactly 1 core and 1GB of RAM allocated to it. Handled perfectly.
- If working from behind a NAT you might also need to setup a [TURN][turn] and/or [STUN][stun] servers. Fortunately, the Media Lab's wired network allocates a public IP to any wired device so the VM used a bridged network and we didn't have to worry about that.
- I later found out that there are some [kurento dockerfiles][kurento_docker] out there for quick containerized deployment of Kurento.

#### Signaling Server

The signaling server is responsible for serving static assets and connecting the clients with the Media server via WebSockets. Our Node.js server was based on Kurento's [one-to-many video call tutorial][one2manytutorial] and ran on my laptop.
When I say *based* I mean completely *copied*. The only change we made was to separate the presenter page from the viewing page and completely remove everything besides the video container on the viewer page. [here's the code.](https://github.com/tomerweller/ml-open-stream)

![Viewer](/assets/open-streaming/view-screenshot.jpg)
![Viewer](/assets/open-streaming/view-screenshot2.jpg)
<small>Screenshots of the live viewing page. Notice the change in the favicon.</small>

Notes: 

- The fun thing about serving files directly from your computer is that you can hot swap them. We were tweaking the viewing experience during the stream, including changing the favicon and various css fixes.
- To avoid exposing my personal laptop (signaling was running directly on my computer, not on a VM) we used the [ngrok][ngrok] tunneling service.

![Terminal](/assets/open-streaming/terminal.png)
<small>left: ngrok with 5 open connection. right: printout from the signaling server.</small>  

#### Presenter Stream
[Tal][tal] managed to find a cheap video capture card in the lab (something [like this][video_capture]) and a laptop that would actually work with it. We used Firefox to connect to our presenter endpoint which handled the WebRTC handshake with the Media server.

Notes:

- Halfway through the stream I realized that there was no audio feed connected to the capture card and that we're actually using the laptop's microphone. Surprisingly, the sound was fairly good.
- Of course, it's far from being a secure solution - anyone could connect to that endpoint and serve as a presenter.


### Results
We had a peak of 37 concurrent streams and averaged around 12. Due to time constraints and because the panel was actually quite interesting we didn't have any continuous monitoring or collected information about load but it sure felt like we could handle a lot more. The machine was responsive (even the VM) throughout the session and I'm still amazed that a tiny VM with no optimizations handled it. KUDOS to [Kurento][kurento].

### Some take aways

1. **When you wait until the last possible minute to do something, it will take exactly one minute.** ([Parkinson's Law][parkinson]). If we had decided to do this a week before then it would have taken a week. Of course, the client would be more polished and performance would be tested. But who cares? Sometimes good enough is good enough.
2. **There's a tool for that.** I always thought that setting up live streaming is a lengthy, non-trivial task. We live in an amazing time where the tools are just there and the only thing that limits us is our perspective. 
3. **You are a server.** In the early days of the internet it was clear that all nodes in the network are created equal. Anyone can consume services and anyone can provide services, whether it's a Web page or an IRC node. The differentiation between a server and a client is a networking protocol technicality. There is no reason why a node can't serve both as a client and a server. However, with massive data centers, Cloud computing and NAT, we've adopted a model of complete separation, our personal machines are clients and those behind some abstract cloud are servers. Serving anything from your machine, not just for debugging purposes, is part of the spirit of the internet. We should build tools that allow it (thanks [ngrok][ngrok]!) and services that take advantage of it. 

Thank you [Tal Achituv][tal] for reminding me all of the above.

[tal]: https://twitter.com/achituv
[kurento]: http://www.kurento.org/
[one2manytutorial]: http://www.kurento.org/docs/6.1.1/tutorials/node/tutorial-3-one2many.html
[kurento_docker]: https://github.com/Kurento/kurento-docker
[stun]: https://en.wikipedia.org/wiki/STUN
[turn]: https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT
[ngrok]: https://github.com/inconshreveable/ngrok
[stallman]: https://stallman.org/
[parkinson]: https://en.wikipedia.org/wiki/Parkinson%27s_law
[joi]: http://joi.ito.com/
[video_capture]: http://usefullinkage.blogspot.com/2007/09/video-capture-card-sale-penny-vidful.html
[gstreamer]: https://gstreamer.freedesktop.org/