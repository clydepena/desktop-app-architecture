# Memory

Where everything lives, and how to get it and give it back without stutters, crashes or slow growth.

Every service depends on memory, and several rules elsewhere in these documents are really memory rules: the audio
thread never allocates, per-frame data is double- or triple-buffered, voices and network messages come from pools,
handles carry generations so freed slots are detected, closing a scope frees everything in it. The memory service's
job is to make memory **fast to get, safe to give back, predictable, and visible**.

It has the familiar shape (a platform layer, strategies over raw memory, budgets, a narrow interface, the same
dependency direction). Its special problems are **lifetime and cost**:

- **Lifetime:** who frees this, and when? Too early crashes; never leaks.
- **Cost:** general-purpose allocation is slow, takes locks and fragments memory over long sessions; in
  garbage-collected languages, allocating garbage causes **collection pauses**, which show up as stutters.

## The layers inside memory management

```
+--------------------------------------------------------------------+
| Consumers          every service: rendering, audio, networking,      |  owned by the app
|                    resources, documents, undo, UI, gameplay           |
+--------------------------------------------------------------------+
| Budgets and        categories, budgets per platform, tracking,       |  visibility
| tracking           high-water marks, leak reports, pressure events    |
+--------------------------------------------------------------------+
| Ownership and      owners, scopes, handles with generations,          |  safety
| lifetime           reference counting, deferred destruction           |
+--------------------------------------------------------------------+
| Allocation         general heap, arenas (per frame), pools, stacks,   |  speed and predictability
| strategies         ring buffers, per-thread allocators                |
+--------------------------------------------------------------------+
| Kinds of memory    CPU heap, GPU memory, memory-mapped files,         |
|                    native memory outside a managed heap, shared        |
+--------------------------------------------------------------------+
| Platform           virtual memory, pages, reserve and commit,         |  platform layer
|                    alignment, memory pressure notifications           |
+--------------------------------------------------------------------+
| OS, hardware (CPU caches, RAM, VRAM)                                    |
+--------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform and virtual memory](01-platform-and-virtual-memory.md) | pages, reserve and commit, alignment, memory pressure, caches |
| 2 | [Lifetimes](02-lifetimes.md) | the central idea: group memory by how long it lives |
| 3 | [Allocation strategies](03-allocation-strategies.md) | heap, arenas, frame arenas, pools, stacks, ring buffers, per-thread allocators |
| 4 | [Garbage-collected languages](04-garbage-collected-languages.md) | allocation rate and pauses, native resources, off-heap memory, leaks by reference |
| 5 | [Ownership and safety](05-ownership-and-safety.md) | owners, reference counting, handles with generations, deferred destruction, scoped cleanup |
| 6 | [Layout and performance](06-layout-and-performance.md) | contiguous data, structure of arrays, hot and cold data, false sharing |
| 7 | [Budgets, tracking and pressure](07-budgets-tracking-and-pressure.md) | categories, budgets, tracking, running short |
| 8 | [GPU memory](08-gpu-memory.md) | video memory budgets, residency, streaming, unified memory |
| 9 | [Using the memory service](09-using-the-memory-service.md) | the public interface, and how each part of an app uses it |
| 10 | [Coupling: what may depend on memory](10-coupling.md) | safe and unsafe dependencies, allocation assertions, soak tests |
| 11 | [Assets](11-assets.md) | budgets, pool capacities, arena sizes, streaming budgets |

## How memory compares

| | Rendering | IO | Time | Memory |
|---|---|---|---|---|
| Role | output | storage | the heartbeat | **where everything lives** |
| Platform layer | GPU APIs | file systems | OS clocks | **virtual memory, pages** |
| Driven by | the main loop | requests | itself | **allocation and release, plus frame and scope boundaries** |
| What goes wrong | dropped frames | hitches | stutter | **crashes (use after free, out of memory), leaks, stutter (pauses, locks)** |
| Abstraction | proxies, handles | logical paths | clocks | **allocators by lifetime, pools, handles with generations** |
| Skipping work | culling | caching | budgets | **reuse: pools, arenas, no steady-state allocation** |
| Test version | headless | in-memory files | manual clock | **tracking and capped allocators** |

## Patterns in memory management

| Pattern | Where |
|---|---|
| Object Pool | fixed-size slots reused for objects of one type |
| Flyweight | one shared copy of data used by many |
| Arena (region-based allocation) | allocate freely, free everything at once |
| Double buffering / ring buffers | data still in use by another frame or thread |
| RAII / dispose | scoped cleanup that can't be forgotten |
| Reference counting | shared data freed by its last user |
| Handles with generations | detecting use of freed slots |
| Deferred destruction | freeing only when nothing can still read it |
| Strategy | allocators behind one interface |
| Decorator | tracking allocators wrapping real ones |
| Observer | memory pressure events |
| Data-oriented design | structure of arrays, hot and cold splitting |

## Related

- [Threads and synchronization](../threads/README.md): per-thread allocators, false sharing and freeing memory other threads may still read.
- [GPU resources](../rendering/02-gpu-resources.md): handles, pools and deferred deletion on the GPU side.
- [Threads and timing](../audio/05-threads-and-timing.md): why the audio thread never allocates.
- [Scripting](../scripting/README.md): behavior added by designers, modders and users, through bindings and sandboxes.
- [Services and scopes](../02-services-and-scopes.md): scopes are where memory lifetimes come from.
- [Resources](../07-resources.md): caches with budgets and eviction.
