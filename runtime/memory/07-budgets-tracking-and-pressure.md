# Budgets, tracking and pressure

Memory that nobody watches grows until something breaks. This layer makes it visible and gives it limits.

## Categories

Every allocation is tagged with what it is for:

| Category | Typical owners |
|---|---|
| textures, meshes, render targets (GPU) | rendering |
| audio clips and streams | audio |
| level / world data | the domain, streaming |
| UI, fonts | the UI toolkit |
| scripts | scripting |
| network buffers | networking |
| undo history, documents | the tool framework |
| caches | resources, IO |

Tags are set per allocator (a pool or arena belongs to one category) or around a block of work, so callers rarely tag
individual allocations.

## Budgets

Each category gets a **budget**, per platform: a console, a phone and a desktop get different numbers. Budgets are a
design decision made early ("textures may use 1.5 GB on this platform"), so content can be made to fit, instead of
discovered late when the product runs out of memory.

## Tracking

- **live counters** per category, shown in a debug overlay or panel
- **high-water marks:** the most ever used, which is what decides whether a budget is safe
- **snapshots** that can be compared: "what grew between entering and leaving this level?"
- **leak reports** at shutdown and at the end of each test: every category should return to its starting level
- **allocation counts per frame**, to check that the steady state allocates nothing

## When memory runs short

Respond in order, from cheapest to most visible:

1. **evict caches** (least recently used first): thumbnails, decoded data that can be reloaded
2. **lower quality:** stream lower texture detail, fewer particles, shorter undo history
3. **refuse a load** with a clear message, before starting it, instead of crashing halfway through

OS **memory pressure** events ([Platform and virtual memory](01-platform-and-virtual-memory.md)) trigger the same steps.
Systems that hold reclaimable memory register to be told, rather than the memory service knowing what they hold
(the **Observer** pattern).

## Running out is a design case

An application that crashes when an allocation fails loses the user's work. Important paths (saving, recovery) keep a
small **reserve** that is released when memory runs out, so they can still complete.
