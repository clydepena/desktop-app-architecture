# Events

Systems announce that something happened; other systems react, without the announcer knowing who listens.

```
documents.saved -> title bar removes "*", recent list updates, recovery copy deleted
selection.changed -> inspector shows the new item, outline scrolls to it
settings.changed("theme") -> UI restyles
file.changedOnDisk -> resource reloads, document asks "reload?"
```

## Three mechanisms

| Mechanism | What | Good for |
|---|---|---|
| **Direct listeners** (observer) | an object keeps a list of listeners and calls them | one source, its own events: a document's `changed`, a selection's `changed` |
| **Event bus** (publish/subscribe) | a shared channel; publishers and subscribers only know the event type | app-wide announcements between unrelated systems |
| **Observable properties** | a value that notifies when set; views bind to it | retained-mode UIs (widgets that persist and must be updated) |

Immediate-mode UIs (redrawn from the data every frame) need far fewer events: they read the current state each
frame. They still need events for things that must *happen* once (reload a resource, scroll to an item).

## Rules

- **Events say what happened, not what to do.** `documentSaved`, not `updateTitleBar`. Listeners decide what it
  means for them.
- **Unsubscribe.** Every subscription is tied to a scope ([Services and scopes](02-services-and-scopes.md)) and removed when the scope
  closes. Forgotten listeners keep dead objects alive and run on closed documents.
- **Deliver on the main thread** by default. Events raised by background jobs are queued and delivered in the main
  loop ([Jobs](06-jobs.md)).
- **Avoid chains.** A listener that publishes an event that triggers another listener that... becomes impossible
  to follow. Keep reactions local; if a sequence matters, write it as a sequence.
- **Don't rely on listener order.** If order matters, it isn't an event, it's a pipeline.
- **Queued or immediate?** Immediate delivery is simpler to debug; queued delivery avoids reacting to half-finished
  changes (several properties set in a row). Queue by default for app-wide events; deliver document-level changes
  after the whole command has run.
