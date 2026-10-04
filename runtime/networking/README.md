# Networking

How a networking service is built, and how the rest of an application uses it without being tied to it.

Networking has the same overall shape as [rendering](../rendering/README.md) and [audio](../audio/README.md): a
platform layer, a stack of layers that each turn something high-level into something lower-level, proxies, a
narrow public interface, and the same dependency direction. Two things make it different:

- **It is two services in one.** **Request/response services** (HTTP: update checks, accounts, cloud saves,
  telemetry, downloads) and **real-time sessions** (multiplayer, live collaboration, a live link between an editor
  and a running app) share the bottom layers but are built very differently above them.
- **The other side is unreliable and untrusted.** Messages arrive late, out of order or not at all, and the remote
  end may be a cheater or an attacker. Rendering and audio talk to your own hardware; networking talks to someone
  else's machine.

## The layers inside a networking service

```
+---------------------------------------------------------------+
| Scene / domain     game objects, documents, accounts            |  owned by the app
+---------------------------------------------------------------+
|        replicate / call / request                               |
+---------------------------------------------------------------+
| Replication        networked objects, network ids, authority,   |  real-time sessions
| and RPC            relevancy, remote calls                      |
+---------------------------------------------------------------+
| Prediction and     client prediction, interpolation, lag        |  real-time sessions
| timing             compensation, ticks, clock sync              |
+---------------------------------------------------------------+
| Session            connect, authenticate, peers and roles,      |  both
|                    timeouts, reconnects                         |
+---------------------------------------------------------------+
| Messages           message types, serialization, versioning,    |  both
|                    compression, delta encoding                  |
+---------------------------------------------------------------+
| Transport          reliable/unreliable channels over UDP, or    |  both
|                    TCP / HTTP / WebSocket; encryption; NAT       |
+---------------------------------------------------------------+
| Platform           sockets (TCP/UDP), DNS, TLS, HTTP client     |  platform layer
+---------------------------------------------------------------+
| OS network stack, network hardware                              |
+---------------------------------------------------------------+
```

Request/response services stop at the session and message layers: a request is "send one message, get one
reply". Real-time sessions use every layer.

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform and transport](01-platform-and-transport.md) | sockets, TCP vs UDP, channels, reliability, NAT traversal, the network thread |
| 2 | [Messages and serialization](02-messages-and-serialization.md) | schemas, versioning, compact encoding, delta encoding |
| 3 | [Sessions and topologies](03-sessions-and-topologies.md) | handshakes, roles, client-server, peer-to-peer, lockstep, liveness |
| 4 | [Replication and RPC](04-replication-and-rpc.md) | network proxies: replicated objects, relevancy, authority, remote calls |
| 5 | [Prediction and timing](05-prediction-and-timing.md) | ticks, clock sync, prediction, reconciliation, interpolation, rollback |
| 6 | [Request/response services](06-request-response-services.md) | HTTP services: async requests, retries, idempotency, tokens, offline mode |
| 7 | [Collaboration and live link](07-collaboration-and-live-link.md) | several people editing one document; an editor connected to a running app |
| 8 | [Security](08-security.md) | never trust the client, authentication, bounds, untrusted input |
| 9 | [Using the network service](09-using-the-network-service.md) | the public interface, and how each part of an app uses it |
| 10 | [Coupling: what may depend on networking](10-coupling.md) | safe and unsafe dependencies, offline mode, loopback, simulated conditions |
| 11 | [Assets](11-assets.md) | the configuration and schemas each layer uses, and what is a secret instead |

## Rendering vs audio vs networking

| | Rendering | Audio | Networking |
|---|---|---|---|
| Platform layer | RHI over GPU APIs | device layer over OS audio APIs | sockets, TLS, HTTP |
| Driven by | the main loop (push) | the device callback (pull) | message arrival, plus fixed ticks |
| Missing a deadline | a dropped frame | a click or dropout | lag, rubber-banding, desync |
| Proxies | render proxies | emitters, instances, listeners | replicated objects, remote proxies for RPCs |
| Skipping work | culling | virtual voices | relevancy (interest management) |
| Budget | frame time, GPU memory | voices, CPU per buffer | bandwidth per client, latency |
| Designer data | materials, shaders | sound events, snapshots | replication settings, schemas |
| Thread rule | hand over, don't share | hand over, never block | hand over, never block on IO |
| The other side | your GPU | your sound card | someone else's machine, untrusted |
| Null version | draws nothing | plays nothing, or mixes offline | offline mode, loopback transport |

## Patterns in networking

| Pattern | Where |
|---|---|
| Bridge / Adapter | the platform layer: one interface over OS socket and TLS APIs |
| Remote Proxy | RPCs: a local object whose calls run on another machine (one of the proxy kinds the Gang of Four describe) |
| Proxy (replica) | a client's copy of a server-owned object, updated by replication |
| Command | messages and RPCs as serialized requests; collaborative edits as serialized undo commands |
| Producer / consumer | the network thread produces received messages, the main thread consumes them |
| Observer / events | connected, disconnected, peer joined, request failed |
| Memento | snapshots for prediction, reconciliation and rollback |
| Strategy | interchangeable transports (UDP, WebSocket, loopback) and topologies |
| Retry with backoff, circuit breaker | request/response services under failure |
| Null object | offline mode: every request fails fast, sessions are local |

## Related

- [Time and scheduling](../time/README.md): clocks, the frame loop, fixed steps and update order that every service runs on.
- [Memory](../memory/README.md): lifetimes, allocators, pools, budgets and GPU memory under every service.
- [Input](../input/README.md): the runtime service that carries information in, from devices to intentions.
- [IO](../io/README.md): files, the virtual file system, packages and streaming, under every service's assets.
- [Scripting](../scripting/README.md): behavior added by designers, modders and users, through bindings and sandboxes.
- [Jobs](../06-jobs.md): asynchronous requests and their completion on the main thread.
- [Data description](../08-data-description.md): which fields exist and how they're encoded; replication and
  message encoding are generated from it.
- [Undo](../../tool-framework/04-undo.md): commands that are also the messages of collaborative editing.
