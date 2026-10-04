# Replication and RPC

How shared state stays in sync across machines. This is where networking has its own **proxies**, in two classic
forms: replicated objects, and remote procedure calls.

## Replicated objects

Each networked object in the application gets a **network record**:

- a **network id** that means the same object on every machine
- which fields replicate (from the data description: [Data description](../08-data-description.md))
- who has **authority** over it
- for each client, the last state that client acknowledged

On the receiving side, the incoming state updates a **local copy** of the object. That copy is a proxy for the
authoritative object on the server: it looks and behaves like the real thing, but the truth lives elsewhere.
(Some engines call these copies "ghosts".)

### What the server does every network tick

1. **Relevancy (interest management):** decide which objects matter to which client: within some distance, visible
   to them, on their team, in the part of the document they have open. This is the network counterpart of culling
   in rendering and virtual voices in audio: don't send what the client can't perceive. In big worlds it is
   essential; it also stops clients learning about things they shouldn't (wall hacks).
2. **Prioritize** within a **bandwidth budget** per client: nearby and fast-changing objects first, distant or
   static ones less often. Objects not sent this tick accumulate priority, so everything is eventually updated.
3. **Send deltas:** only the fields that changed since the state that client acknowledged
   ([Messages and serialization](02-messages-and-serialization.md)).
4. **Spawn and despawn:** objects entering a client's relevancy are created there; leaving it, destroyed there.

## Authority

For each object, exactly one side decides its truth:

- usually the **server** (it validates everything)
- sometimes the **owning client** for things it directly controls (its own character's movement, for
  responsiveness), with the server checking that the results are plausible
- authority can be **transferred** (a player picks up an object and now controls it)

Without a clear authority rule, two machines fight over the same value and it flickers between them.

## Network ids

Pointers and local indexes mean nothing on another machine. Objects are referred to by **network ids**: stable,
assigned by the authority, and mapped to local objects on each machine. These are the same kind of stable ids that
undo, saving and linking between documents need; a design with stable ids from the start makes all four easier.

## Remote procedure calls: the Remote Proxy

The Gang of Four's **Remote Proxy** pattern is exactly this: a local object whose method calls are sent to a real
object on another machine.

```
server.requestPickup(itemId);      // called on a client, serialized, sent, executed on the server
allClients.playHitEffect(pos);     // called on the server, executed on every client
owner.showMessage("Level up");     // called on the server, executed on the owning client only
```

RPCs are **asynchronous**: they return immediately. Results come back as later replicated state or another
message, never as a return value to wait for. Calling code must be written for "this will happen later, or maybe
never".

### Replication or RPC?

| Use replication for | Use RPCs for |
|---|---|
| state: positions, health, door open or closed | one-off events: play an effect, show a message |
| anything a client joining later must see | anything that only matters at the moment it happens |
| values that change often | requests from a client: "I want to do X" |

A door opened by RPC stays closed for a player who joins afterwards; a door whose `open` field replicates is
correct for everyone.
