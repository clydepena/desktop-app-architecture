# Coupling: what may depend on networking

The same direction rule as rendering and audio: **upper layers may depend on the network service's interface; the
network service depends only on what is below it** (sockets, jobs or asynchronous IO, the serialization generated
from the data description, logging). Anything flowing upward arrives as events and received messages, dispatched on
the main thread.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle** | starts, pumps and shuts down the service: the single owner |
| **Platform sockets, TLS, HTTP** | below the service |
| **Data description** | the network's encoding reads it; the description doesn't depend on the network |
| **Bridges / networked components** | connecting objects to replication and network ids is their purpose |
| **Account, matchmaking and lobby UI** | the application layer, above request/response services |
| **Editor multiplayer testing, simulators, profilers, collaboration** | the tool layer sits above the network service |

## Keep decoupled

| System | How it reaches the network instead |
|---|---|
| **Domain logic** | sends intentions through RPCs, reads replicated state; never builds packets or waits for replies |
| **Rendering and audio** | never touch the network; they present domain state, which replication keeps up to date |
| **Undo and documents** | stay local. With collaboration, serialized commands become messages through a collaboration layer; undo itself doesn't know peers exist |
| **Persistence** | separate from replication, even when both reuse the data description: a save file is not a network snapshot |
| **Physics** | runs inside the simulation; prediction and rollback snapshot and restore it through the domain, not through network code |
| **Input** | becomes input messages at the domain or bridge level; the network never reads devices |

## Mistakes to avoid

- **Blocking calls on the main thread:** connecting, DNS lookups, waiting for an HTTP reply.
- **Assuming a reply arrives instantly, or at all:** every remote call needs a timeout and a "never came back" path.
- **Trusting client-sent results** instead of validating intentions ([Security](08-security.md)).
- **Sending pointers, local indexes or whole objects** instead of network ids and described fields.
- **Tying the wire format to class layouts without versioning**, so one renamed field breaks every older client.
- **Network calls scattered through gameplay code** instead of going through the replication and RPC doorways.
- **Global access to "the connection"** from anywhere, which hides who depends on it
  ([Services and scopes](../02-services-and-scopes.md)).

## Testing the boundary

- **Offline mode / null network** (the **Null Object** pattern): the application runs fully offline; requests fail
  fast with "offline", sessions are local. If everything still works (single-player, editing, saving, tests), the
  decoupling holds.
- **Loopback transport:** server and clients in one process, connected through in-memory queues instead of sockets.
  Replication and RPCs can be tested deterministically and fast.
- **Simulated conditions:** run the test suite with 200 ms of latency, jitter and 5% packet loss.
- **Record and replay:** record a session's incoming messages and replay them to reproduce a bug exactly.
