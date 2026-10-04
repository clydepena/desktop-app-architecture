# Using the network service

How the rest of an application uses the network. As with rendering and audio, application code describes **what**
it wants (this object is shared, I intend to do this, fetch this resource) through a few doorways, and the service
handles packets, retries and timing. Everything coming back (messages, replicated state, connection changes) is
delivered on the main thread, at a fixed point in the frame or tick.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Requests** | request/response services | `http.get(url) -> Future<Response>`, `post(url, body)`, with timeout, retry and cancel |
| **Session** | host, join, leave | `host(port, settings)`, `connect(address, token) -> Future<Session>`, `disconnect()` |
| **Messages** | typed messages on channels | `send(peer, message, Channel.RELIABLE)`, `on(MessageType, handler)` |
| **RPC** | remote calls through remote proxies | `server.requestPickup(id)`, `allClients.playEffect(position)` |
| **Replication** | register networked types and objects | `registerReplicated(Type, description)`, `spawn(object) -> NetworkId`, `setAuthority(object, peer)` |
| **Relevancy and priority** | tune what each peer receives | `setRelevancy(object, rule)`, `setPriority(object, p)` |
| **Time** | ticks and the shared clock | `currentTick()`, `serverTime()`, `roundTripTime(peer)` |
| **Events** | what happened | `connected`, `disconnected(reason)`, `peerJoined`, `authorityChanged`, `requestFailed` |
| **Diagnostics** | health of the connection | ping, packet loss, bandwidth per channel, replication counts |

## How each part of an application uses it

### The lifecycle owns and pumps the service

The main loop ([Lifecycle](../01-lifecycle.md)) starts the service, takes incoming messages at a fixed point each
frame or tick, and at shutdown disconnects politely (with a timeout) and cancels pending requests.

### The data description drives encoding

Which fields replicate, how they are encoded and in what ranges come from the data description
([Data description](../08-data-description.md)). Network code doesn't hand-write serializers per class.

### The domain goes through bridges

As with rendering and audio, a **networked component** (a bridge) registers its object for replication and maps
it to its network id. Gameplay code:

- calls RPCs with **intentions** ("request pickup")
- reacts to **replicated state** and events
- never touches sockets, packets or byte buffers

### Rendering and audio stay unaware

They present the **replicated domain state** like any other state. Interpolation and prediction happen in the
domain or bridge layer, before rendering sees the positions.

### Settings and secure storage

Server addresses, ports, regions and bandwidth limits are settings ([Settings](../04-settings.md)). Credentials and
tokens go to the operating system's secure storage, never to settings files.

### Diagnostics

Ping and packet-loss overlays, connection events in the log ([Diagnostics](../05-diagnostics.md)), and a **network
profiler** showing bandwidth per object type and per field, so the expensive data is visible.

### Editors

- **Multiplayer testing:** a server and several clients in one editor process (each with its own world), or several
  local processes started together.
- **A network condition simulator:** add latency, jitter, packet loss and bandwidth limits on purpose. Bugs that
  only appear at 150 ms of latency are invisible on a local connection.
- **Replication debugging:** which objects each client sees, who has authority, when each was last updated.
- **Collaboration and live link** ([Collaboration and live link](07-collaboration-and-live-link.md)).

## The same shape as rendering and audio

Doorways, proxies, bridges and events, again. The differences: the remote side is unreliable and untrusted, and
calls are asynchronous by nature, so nothing in the application may wait for an answer.
