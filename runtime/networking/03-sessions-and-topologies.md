# Sessions and topologies

A **session** is a group of connected peers sharing something: a match, a collaborative document, a live link.
This layer handles who is connected, in which role, and what happens when connections fail.

## Connecting

```
1. transport connection (with encryption)
2. handshake: protocol version check, then authentication (a token from an account service)
3. role assigned: server, client, host, observer
4. initial state sent (the world, the document), then live updates
```

Connecting is **asynchronous** with progress, a timeout and a Cancel button. "Connecting..." must never freeze the
app.

## Topologies

| Topology | How it works | Used for | Trade-offs |
|---|---|---|---|
| **Client-server, authoritative** | the server owns the true state; clients send input, receive state | most online games, collaborative editors | needs servers; the server's decision is final, which makes cheating hard |
| **Listen server / host** | one player's machine is also the server | small co-op games | no server cost; the host has an advantage, and the session ends if the host leaves (unless the host migrates) |
| **Peer-to-peer** | everyone sends to everyone | small sessions, voice | no server; bandwidth grows with the number of peers; no natural authority |
| **Deterministic lockstep** | everyone runs the same simulation and exchanges only inputs | strategy games with thousands of units, fighting games (with rollback) | tiny bandwidth; needs bit-identical simulation on every machine, which is hard (floating point, randomness, iteration order) |

The choice drives everything above: replication exists in client-server; lockstep replaces it with input exchange.

## Liveness

- **Heartbeats:** small messages in both directions, so silence can be detected.
- **Timeouts:** no message for N seconds means the peer is gone. Pick N to tolerate brief hiccups (a train tunnel)
  without leaving dead peers in the session for long.
- **Disconnect reasons:** "kicked", "server shutting down", "version mismatch", "timed out", each reported to the
  application as an event, so the UI can say something useful.
- **Reconnecting:** a client that drops can rejoin the same session (same identity, same objects) within a grace
  period, instead of starting over.

## Above sessions

**Lobbies, matchmaking, server browsers and friends lists** are usually separate **request/response services**
([6](06-request-response-services.md)): they find or create a session and hand over its address and a join token.
The session layer only starts from there.
