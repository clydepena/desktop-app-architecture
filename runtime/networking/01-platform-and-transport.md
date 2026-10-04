# Platform and transport

The bottom two layers: talking to the operating system's network stack, and turning raw packets into channels
the rest of the service can rely on.

## The platform layer

What the operating system provides, behind one interface (the **Bridge / Adapter** pattern):

- **Sockets:** TCP (reliable streams) and UDP (independent datagrams)
- **DNS:** turning names into addresses
- **TLS:** encryption and certificate checking
- **An HTTP client** and **WebSockets**, often from a library

**The one rule: non-blocking or asynchronous only.** A blocking `connect`, `read` or DNS lookup on the main thread
freezes the application for as long as the network takes, which can be seconds, or forever if a firewall silently
drops packets.

## TCP or UDP

| | TCP | UDP |
|---|---|---|
| Delivery | reliable, ordered | may be lost, duplicated or reordered |
| One lost packet | delays everything behind it until resent (head-of-line blocking) | affects only itself |
| Good for | HTTP, file transfer, chat, editor collaboration, live links | fast real-time games, voice |

Many real-time apps use UDP and build exactly the reliability they need on top. Modern protocols such as QUIC
provide that ready-made (several independent streams over UDP, with encryption).

## Channels

A transport over UDP usually offers several **channels**, each with its own guarantees:

| Channel | Guarantee | Typical messages |
|---|---|---|
| **Unreliable** | none: a lost message stays lost | positions every tick (the next one replaces it anyway) |
| **Unreliable, sequenced** | old messages are dropped if a newer one already arrived | continuous state where only the latest matters |
| **Reliable, unordered** | arrives eventually, in any order | events: "picked up item 42" |
| **Reliable, ordered** | arrives eventually, in order | chat, a sequence of RPCs that depend on each other |

To provide them, the transport adds:

- **sequence numbers and acknowledgements** (often a bitfield acknowledging the last 32 packets at once)
- **resending** of reliable messages that weren't acknowledged in time
- **fragmentation:** splitting messages larger than one packet (about 1,200 bytes is a safe size on the internet)
  and reassembling them
- **congestion control:** sending less when the connection shows loss, instead of flooding it
- **connection ids and encryption**, so packets from strangers are rejected cheaply

## NAT traversal and relays

Most home and mobile devices sit behind routers (NAT) that don't accept incoming connections. Two machines that
both sit behind NAT can't simply connect to each other. The usual solutions:

- **A rendezvous server** that both contact; it tells each the other's public address, and both send packets at
  the same time to open a path ("hole punching").
- **A relay server** that forwards traffic when hole punching fails. Always available, at the cost of extra latency
  and server bandwidth.
- **Dedicated servers** with public addresses avoid the problem for client-server games.

## The network thread

Sockets are serviced by a **network thread** (or by asynchronous IO): it receives packets, sends acknowledgements,
resends what was lost, and queues **complete messages** for the main thread. The main loop takes them at a fixed
point in each frame or tick, the same **producer / consumer** hand-over as render and audio threads. Outgoing
messages go the other way through a queue.

Networking is **soft real-time**: a late packet is absorbed by prediction and interpolation
([5](05-prediction-and-timing.md)), not a hard failure like an audio underrun. But the thread must still never
block the main thread, and the main thread must never block on it.
