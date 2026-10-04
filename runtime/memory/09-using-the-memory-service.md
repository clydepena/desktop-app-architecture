# Using the memory service

How the rest of an application gets memory: by choosing the strategy that matches each lifetime, and tagging what it
is for. The memory service never knows what memory is used for, except through those tags.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Frame memory** | temporary data for this frame | `frameArena().alloc(size)`, reset automatically at the frame boundary |
| **Scratch memory** | temporary data for one task | `scratch() -> Arena`, freed when the task ends |
| **Pools** | many objects of one type | `pool<T>(capacity)`, `pool.acquire()`, `pool.release(item)` |
| **Scoped memory** | memory that lives as long as a scope | `scope.arena()`, freed when the scope closes |
| **Tagging** | attribute allocations to a category | `withCategory("Textures", () -> ...)`, or a category on each allocator |
| **Budgets** | limits per category | `budget("Textures")`, `isOverBudget(category)` |
| **Statistics** | what is used where | usage per category, high-water marks, `snapshot()`, `diff(a, b)` |
| **Pressure** | react to low memory | `onMemoryPressure(level, handler)` |
| **Debug features** (debug builds) | catch mistakes | guard bytes, fill patterns for freed memory, leak reports, `assertNoAllocationsThisFrame()` |

## How each part of an application uses it

| Part | How it uses memory |
|---|---|
| **Lifecycle and time** | reset the frame arena at a fixed point each frame (accounting for frames in flight); create and close scopes with their memory ([Time and scheduling](../time/README.md)) |
| **Rendering** | per-frame upload memory in ring buffers or frame arenas; pools for proxies and handles; its own GPU budget, with streaming to stay under it ([GPU memory](08-gpu-memory.md)) |
| **Audio** | pre-allocates voices, instance slots and stream ring buffers so the audio thread never allocates; clips tagged to the audio category |
| **Networking** | pools for messages and packets; ring buffers for send and receive queues; per-connection limits, since memory is also a security boundary |
| **Input** | small, fixed, pre-allocated state and snapshot buffers |
| **IO** | streaming buffers from pools, memory-mapped archives, scratch arenas for decompression |
| **Resources** | caches with budgets and least-recently-used eviction; the first to respond to pressure |
| **Jobs** | per-thread scratch arenas, freed when each job ends |
| **Undo history** | limited by memory as well as by step count; large commands count against a budget |
| **UI toolkit** | immediate-mode UIs build their draw data every frame in a frame arena |
| **Diagnostics and editors** | a memory profiler panel (usage per category over time, high-water marks, snapshot diffs, top allocators), leak reports at shutdown, allocations per frame in the profiler |
