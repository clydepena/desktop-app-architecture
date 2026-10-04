# Collaboration and live link

Networking for **editors**: several people editing the same document, and an editor connected to a running
application. Both build directly on the tool framework.

## Collaborative editing

### Commands are operations

Each undo command ([Undo](../../tool-framework/04-undo.md)), serialized, is also the message sent to the others:
"set the color of shape 17 from red to blue", "delete layer 4". Two earlier design decisions pay off twice here:

- **commands that store values**, not references to live objects, can be serialized as they are
- **stable ids** let a command name the same object on every machine

### Ordering and conflicts

Two people can change the same thing at the same time. The common solutions, from simplest to most capable:

| Approach | How it works | Good for |
|---|---|---|
| **Locks** | an object being edited is locked for others | coarse editing: one person per object or per file |
| **Last write wins, per property** | the server orders changes; the latest value of each property wins | properties of objects (colors, positions); simple and often good enough |
| **Operational Transformation (OT)** | the server orders operations; each client rewrites ("transforms") its pending operations against the ones that arrived first | text and lists (the Google Docs approach) |
| **CRDTs** (conflict-free replicated data types) | data structures whose operations merge to the same result in any order, even without a central server | offline-first apps, peer-to-peer collaboration |

Most editors use a server that orders operations, plus last-write-wins on properties, and OT or CRDTs only for text
and ordered lists.

### Presence

Other people's cursors, selections and viewports, shown live. Sent as lightweight, **unreliable** updates (the next
one replaces a lost one), never saved with the document and never part of undo.

### Undo becomes per user

Undoing *your* last change must not revert someone else's later change to the same object. Each user's history
holds only their own commands; undoing one sends its inverse as a new operation, which the server orders like any
other.

## Live link

An editor connected to a running application (on the same machine, another computer, or a phone or console):

- the editor sends **property changes** (tweak a value, see it change in the running app immediately)
- the app sends back **logs, statistics and screenshots**, and can be paused, stepped and inspected
- assets can be **pushed** to the running app and hot-reloaded

A live link is request/response plus a small replication channel. It is simpler than collaboration (one editor,
one target, the editor usually has authority) and very valuable for testing on real devices.
