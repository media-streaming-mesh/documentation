
# What Is Media Streaming Mesh?
Media Streaming Mesh is a new concept for supporting real-time applications (such as media production and multiplayer online gaming) in Kubernetes.

The goal of Kubernetes is to provide a "platform for automating deployment, scaling, and operations of application containers across clusters of hosts".  Most applications deployed in Kubernetes today are web-based, and so much of the effort around networking in kubernetes is optimised for web applications.  One example of this is the service mesh architecture (exemplified by [Istio](https://istio.io)), where applications communicate with each other via web proxies rather than directly over IP.

Media Streaming Mesh will enable developers of real time applications to focus on their business logic whilst the Media Streaming Mesh infrastructure facilitates real-time connectivity for microservices.

# The Goals of Media Streaming Mesh

## Extend the benefits of service mesh to real-time applications

Today's service meshes generally only support TCP-based applications (and in fact are optimised for HTTP-based web applications).   Any support for UDP that is added to service meshes is likely to be focussed on enabling [QUIC](https://en.wikipedia.org/wiki/QUIC) (since HTTP/3 runs over QUIC).

Real-time applications generally run over UDP rather than TCP.  Media Streaming applications typically rely on RTP (the [Real-time Transport Protocol](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol)) - which runs on top of UDP, and hence RTP will be the initial focus of Media Streaming Mesh.

Service meshes bring many benefits to web applications such as:

* "Layer 7" URL based routing
* flexible load-balancing
* support for canary deployments
* failure detection
* statistics/metrics export
* pod-to-pod authentication
* data encryption

Our goal is to extend all these to real-time media streaming applications, whilst also enabling additional capabilities such as:

* data protection using [Forward Error Correction](https://en.wikipedia.org/wiki/Error_correction_code#Forward_error_correction), sending data over multiple paths, and/or [NAK](https://en.wikipedia.org/wiki/Acknowledgement_(data_networks))-based mechanisms
* Stream fan-out to multiple clients by replication and/or unicast to multicast conversion.

## Support interactive and streaming applications

Interactive real-time apps (e.g. games) generally use de-facto standard protocols (such as [RakNet](https://en.wikipedia.org/wiki/RakNet), [KCP](https://www.improbable.io/blog/kcp-a-new-low-latency-secure-network-stack) and [netcode](https://github.com/networkprotocol/netcode)) which run over UDP.  UDP itself is connectionless, so to support these protocols we can either rely on timer heuristics etc. or implement per-protocol proxies.

Streaming apps are generally RTP-based as noted above.   RTP enables measurement of loss and jitter as it carries sequence numbers and timestamps in the packet header.

One challenge with RTP is that it often runs on ephemeral UDP ports which are assigned by a TCP-based control channel such as [SIP](https://en.wikipedia.org/wiki/Session_Initiation_Protocol) or [RTSP](https://en.wikipedia.org/wiki/Real_Time_Streaming_Protocol).    However proxying these TCP-based protocols will enable us to implement URL/URI-based routing and to avoid using timer heuristics.

## Enable intra-cluster, inter-cluster & “extra-cluster” (Internet) apps

Many cloud-native applications involve a mixture of 'east-west' traffic between microservices (generally within the same cluster) and 'north-south' traffic between the application and external entities.

This will be equally true for real-time applications.

For example in a game there might be traffic between game players and the game infrastructure running in the cloud.   However for large game instances the game itself might be spread over multiple compute nodes (possibly even distributed geographically), and these will need to communicate with each other.

Equally for media applications there might be multiple camera feeds into a news-room where one feed is selected, various data (e.g. breaking news) is overlaid, and then the resulting stream is sent out for broadcast.


# How Does It Work?

The exact architecture for Media Streaming Mesh is still very much up for discussion.

Our current demo implementation relies on a simple Go-based proxy that runs as a pod sidecar (plus an init container that directs RTSP, RTP and RTCP traffic into the proxy).

Longer term our expectation is that we'll implement:

* [SPIFFE](https://spiffe.io/docs/latest/spiffe-about/overview/)/[SPIRE](https://spiffe.io/docs/latest/spire-about/spire-concepts/) for pod to pod authentication
* A per-node RTP/SRTP proxy (most likely based on [FD.io VPP](https://wiki.fd.io/view/VPP))
* A per-cluster RTSP-proxy

With that baseline we will then be able to implement other protocols (such as SIP, [RIST](https://en.wikipedia.org/wiki/Reliable_Internet_Stream_Transport), [SMTPE 2110](https://en.wikipedia.org/wiki/SMPTE_2110), "raw" RTP etc.)

In order to keep footprint light one key will be to deploy only the required components for the service being implemented.

For inter and extra-cluster traffic the per-node RTP/RTSP proxies will act as data-plane gateways, and the per-cluster proxies will act as control-plane gateways.


# How Can I Get Involved?

We're looking for potential users of Media Streaming Mesh to help us define the solution, and for developers to help us create it!

Please do join our [Slack channel](https://cloud-native.slack.com/app_redirect?channel=media-streaming-mesh).
