# Messages and serialization

What travels over the transport: typed messages, encoded compactly, in a format that both ends agree on even when
they are different versions of the app.

## Typed messages with schemas

Each message is a typed structure, not an ad-hoc string:

```
message PlayerInput { uint32 tick; uint8 buttons; float aimAngle; }
message ChatLine    { string text; }
message Spawn       { NetworkId id; TypeId type; Vector2 position; }
```

The definitions live in one place: an interface definition language (Protocol Buffers, FlatBuffers, Cap'n Proto
are common ones) or the application's own **data description** ([Data description](../08-data-description.md)).
Encoders and decoders are generated from them, never written by hand per message.

## Versioning

Client and server builds may differ, and old clients keep connecting after an update. So:

- every connection starts by exchanging a **protocol version**; incompatible versions are refused with a clear
  message ("please update")
- fields are **added** in compatible ways: new fields have defaults, old readers skip what they don't know
- fields are **never renamed or reused** for a different meaning; removed fields stay reserved

The same idea as file format versions ([Persistence](../../tool-framework/03-persistence.md)), with stricter
consequences: a mismatch disconnects people.

## Compact encoding

Bandwidth is limited and every byte is sent to every client, many times per second. Real-time protocols squeeze:

- **Bit-packing:** a boolean takes 1 bit, a value from 0 to 100 takes 7 bits.
- **Quantization:** a position stored to the nearest centimeter within the world's bounds, an angle in 10 bits,
  instead of full 32-bit floats. The data description's ranges (`Range(min, max)`) tell the encoder how.
- **Compression** of larger messages (snapshots, chat history, documents).

## Delta encoding

Instead of the full state, send only what changed **since a state the receiver has confirmed**:

```
client acknowledged snapshot 120
server builds snapshot 125, compares with 120, sends only the fields that differ
```

Basing deltas on *acknowledged* states (not just the previous message) keeps them correct when packets are lost:
the receiver always has the base the delta refers to.

## Rules

- **Decoding is untrusted input.** Every length, count and id from the network is checked before use
  ([Security](08-security.md)).
- **No pointers or local indexes** on the wire: objects are referred to by network ids
  ([Replication and RPC](04-replication-and-rpc.md)).
- **Separate from save files.** Both may reuse the data description, but a save file and a network message have
  different needs (completeness vs size and speed) and different version rules.
