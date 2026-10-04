# Lifetimes

The central idea: most memory problems disappear when memory is grouped by **how long it lives**, and allocated
accordingly.

| Lifetime | Examples | Natural strategy |
|---|---|---|
| **The whole application** | services, registries, the action list | allocated at startup, freed at shutdown |
| **A scope** (workspace, document, level, world chunk) | loaded assets, a document's data, a level's objects | an arena or pools per scope, freed when the scope closes |
| **One frame** | draw lists, temporary strings, per-frame UI data, query results | a **frame arena**: bump a pointer to allocate, reset the whole arena at the end of the frame |
| **One task or function** | decoding buffers, temporary arrays | a scratch arena or the stack, freed when the task ends |
| **One object, unpredictable** | game objects, editor items | a pool per type, or the general heap |

## Scopes are lifetimes

This matches [Services and scopes](../02-services-and-scopes.md) exactly: a scope owns its services **and** its
memory, and closing it frees both. A workspace scope frees all its caches; a level scope frees all its objects at once,
without walking them one by one.

## "Who frees this?" should be easy

With lifetimes, the answer is almost always "whoever owns the lifetime it belongs to":

- per-frame data: nobody frees it individually; the frame arena is reset
- a level's objects: the level's scope
- a texture shared by several documents: the scope that outlives them all (the workspace), or reference counting
  ([Ownership and safety](05-ownership-and-safety.md))

When the answer is hard, the data probably has an unclear lifetime, and that is the design problem to fix.

## Mixing lifetimes is what fragments memory

A general-purpose heap that holds long-lived and short-lived allocations side by side ends up with small free gaps
between long-lived blocks: **fragmentation**. Memory is free, but not in pieces large enough to use. Applications
running for a long time (an editor open for days, a server running for weeks) suffer most. Separating lifetimes into
their own arenas and pools prevents it.
