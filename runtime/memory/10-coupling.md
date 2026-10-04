# Coupling: what may depend on memory

Memory is the **lowest** runtime service: everyone depends on it. It depends only on the platform layer, and never
knows what memory is for except through the category tags callers give it.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Every service** | allocates through the strategies that fit its lifetimes, and tags its categories |
| **Lifecycle, time, scopes** | reset frame arenas and free scoped memory at the right moments |
| **Resources, rendering** | react to budgets and pressure (eviction, lower quality) |
| **Diagnostics, editor profilers** | read statistics; the tool layer, above memory |

## Keep decoupled

| System | How it handles memory instead |
|---|---|
| **Domain / gameplay** | uses the containers and pools its systems provide; doesn't pick allocators per call or manage raw memory |
| **Between services** | no service frees another's memory or keeps raw pointers into it; they exchange handles, copies or data owned by a shared scope |
| **Saved data** | never contains addresses or allocator details, only values and stable ids |

## Mistakes to avoid

- **Allocating in hot per-frame loops:** heap churn, lock contention, or collector pauses in managed languages.
- **Unbounded caches and histories** that only grow.
- **Leaks through forgotten listeners, timers and static collections**, common in garbage-collected languages.
- **Use after free:** raw pointers kept past their owner's lifetime, especially across a scope close.
- **Freeing memory still used by the GPU or another thread** (no deferred destruction).
- **Freeing with a different allocator** than the one that allocated.
- **Fragmentation in long-running applications** (an editor open for days) from mixing lifetimes in one heap.
- **No plan for running out of memory:** crashing halfway through a load instead of refusing it.
- **False sharing** between worker threads.
- **Allocating on the audio thread.**

## Testing the boundary

- **Allocation assertions:** a tracking allocator fails a test if a steady-state frame allocates from the heap.
- **Leak checks** at the end of each test and at shutdown: every category returns to its starting level.
- **Soak tests:** run for hours (or simulate days of editing) and watch for growth and fragmentation.
- **Low-memory simulation:** cap budgets artificially to test eviction, quality fallbacks and clean failure.
- **Debug allocators:** guard pages and fill patterns turn overruns and use after free into immediate, obvious
  failures.
