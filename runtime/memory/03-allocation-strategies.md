# Allocation strategies

Different lifetimes and access patterns call for different allocators. They share one interface (the **Strategy**
pattern), so code can be handed whichever fits.

| Strategy | How it works | Good for | Cost |
|---|---|---|---|
| **General-purpose heap** (`malloc` / `new`) | finds a free block of any size | anything with unpredictable size and lifetime | slower, may take a lock, **fragments** over long sessions |
| **Arena / linear / bump allocator** | allocates by moving a pointer forward; frees **everything at once** | per-frame data, per-task scratch, loading a level | individual items can't be freed |
| **Double-buffered frame arenas** | one arena per frame in flight | per-frame data that the GPU or a render thread still reads next frame | memory x frames in flight |
| **Pool** (fixed-size blocks, free list) | equal-sized slots, handed out and returned | particles, audio voices, network messages, objects of one type | fixed capacity, chosen in advance |
| **Stack allocator** | an arena that frees in reverse order | nested temporary work | strict order |
| **Ring buffer** | wraps around; the oldest data is consumed or overwritten | streaming (audio, IO), command queues between threads, logs | fixed size; readers must keep up |
| **Per-thread allocators** | each thread has its own arena or cache | job workers | avoids lock contention |

## Frame arenas in detail

```
frame start:   arena.reset()                    (or the arena for this frame-in-flight slot)
during frame:  p = arena.alloc(size)            (a pointer bump: nanoseconds)
frame end:     nothing to free
```

With frames in flight ([Frames and threads](../rendering/05-frames-and-threads.md)), each slot gets its own arena,
reset only when the GPU has finished with that slot's frame.

## Pools in detail

A pool pre-allocates N slots of one size. `acquire` takes a free slot from a list; `release` returns it. No
fragmentation, constant time, and the memory is contiguous (good for iterating). When a pool is full, the caller
decides: refuse (a voice limit), reuse the oldest, or grow by adding another block.

## Two strong rules

- **The audio thread never allocates** ([Threads and timing](../audio/05-threads-and-timing.md)): everything it needs
  is pre-allocated in pools and ring buffers.
- **Steady state allocates nothing:** after startup and loading, a normal frame should make **no** general-heap
  allocations, only arena bumps and pool reuse. Many engines measure this every frame.

## Wrapping allocators

A **tracking allocator** wraps another and counts what passes through it (bytes, calls, categories): the **Decorator**
pattern. The same trick adds debug checks (guard bytes, fill patterns) in debug builds without changing callers.
