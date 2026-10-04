# Layout and performance

The same amount of memory can be fast or slow depending on how it is arranged. Because the CPU reads memory in cache
lines and keeps recently used ones close ([Platform and virtual memory](01-platform-and-virtual-memory.md)), **how**
data is laid out often matters more than how much work is done on it.

## Contiguous arrays beat linked structures

Walking an array reads memory in order: each cache line loaded is fully used, and the hardware prefetches the next
ones. Following pointers (linked lists, trees of separately allocated objects) jumps around memory and misses the cache
on almost every step. For data processed every frame, store it in arrays.

## Array of structures vs structure of arrays

```
array of structures:   [x y z vx vy vz alive] [x y z vx vy vz alive] ...
structure of arrays:   x[] y[] z[] vx[] vy[] vz[] alive[]
```

| Layout | Good when |
|---|---|
| **Array of structures** | every field of an item is used together |
| **Structure of arrays** | a loop touches a few fields of many items (updating positions reads only x, y, z, vx, vy, vz) |

Structure of arrays is the core idea of **data-oriented design**. It is why render proxies, entity-component-system
component arrays, particle systems and audio voice arrays are stored the way they are: the hot loop loads only the
fields it needs, densely packed, and can use SIMD instructions.

## Hot and cold data

Fields used every frame (position, velocity) are kept apart from fields used rarely (a display name, editor-only
notes, debug data). The hot loop then loads less memory per item. Editor-only data in particular should not sit in the
middle of runtime structures.

## False sharing

Two threads writing **different** variables that happen to sit in the **same cache line** slow each other down
dramatically: each write forces the other core to reload the line. Per-thread counters, per-thread arenas and job
results are padded or separated to their own cache lines.

## Fewer, larger allocations

Many small allocations cost time (allocator bookkeeping, locks) and scatter data across memory. Allocating one block
for many items (a pool, an array, an arena) is faster to create, faster to walk, and faster to free.

## Measure first

Memory performance is often counter-intuitive. Profilers that show cache misses and memory bandwidth (not just time
per function) find the real problems; layout changes are then guided by measurements, not guesses.
