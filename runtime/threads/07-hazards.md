# Hazards

The bugs that only concurrency produces. They tend to be rare, timing-dependent and impossible to reproduce on demand,
so they are best prevented by design.

## Data races

Two threads access the same memory at the same time, at least one of them writes, and nothing orders the accesses.
Results: torn values, lost updates, corrupted containers, crashes far from the cause. In some languages (C, C++) a data
race makes the whole program's behavior undefined.

**Prevention:** one owner per piece of data, hand-over instead of sharing, and locks or atomics where sharing is
unavoidable ([Handing data between threads](04-handing-data-between-threads.md)).

## Race conditions in logic

Even with every access locked, the **sequence** can be wrong: "check, then act" across two separate locked calls.

```
if (!cache.contains(id))     // another thread can insert between these two lines
    cache.insert(id, load(id));
```

**Prevention:** make the whole decision one operation (`getOrInsert`), under one lock or one atomic.

## Deadlocks

Thread 1 holds lock A and waits for B; thread 2 holds B and waits for A. Both wait forever.

**Prevention:**

- a global **lock order**: whenever two locks are needed, always take them in the same order
- never call **unknown code** (callbacks, events, plugins) while holding a lock, since it may take other locks
- keep locks **private** to one class, so the order can be checked in one place
- use timeouts in debug builds to turn a silent hang into a report showing which threads hold which locks

Deadlocks also happen **without locks**: two threads each waiting for the other's message, or a task pool where every
worker waits for a task that can't start ([Task systems](06-task-systems.md)).

## Priority inversion

A high-priority thread waits for a lock held by a low-priority thread, which can't run because medium-priority threads
keep the CPU busy. The high-priority thread effectively runs at low priority. A famous case reset a spacecraft's
computer repeatedly on Mars in 1997.

**Prevention:** the audio thread and other critical threads take **no locks** shared with lower-priority threads
([Threads and timing](../audio/05-threads-and-timing.md)); some platforms offer mutexes with **priority inheritance**,
which temporarily raise the holder's priority.

## Starvation and livelock

- **Starvation:** a thread never gets its turn (a writer behind endless readers, a low-priority task behind a steady
  stream of high-priority ones).
- **Livelock:** threads keep reacting to each other without making progress, like two people stepping aside in the same
  direction in a corridor.

## False sharing

Two threads write **different** variables that happen to sit on the same CPU **cache line** (typically 64 bytes).
Every write forces the line to bounce between cores, and code that looks independent runs many times slower.

**Prevention:** pad or align per-thread counters and hot fields to cache-line size, or keep per-thread data in separate
allocations ([Layout and performance](../memory/06-layout-and-performance.md)).

## The ABA problem

A lock-free structure reads value A, gets descheduled, and meanwhile another thread changes A to B and back to A (for
example, freeing a node and reusing its memory). The first thread's compare-and-set succeeds, wrongly assuming nothing
changed. **Prevention:** tagged pointers with a counter, hazard pointers or epoch-based reclamation; in practice, use a
tested library ([Atomics and memory ordering](03-atomics-and-memory-ordering.md)).

## Use after free across threads

One thread frees an object another thread is still reading. **Prevention:** deferred destruction (free only after
every thread has passed a known point, such as the end of a frame), reference counting, or handles with generations
([Ownership and safety](../memory/05-ownership-and-safety.md)).

## Oversubscription

More busy threads than cores: several libraries each creating their own pool, or blocking tasks prompting the pool to
add threads. The OS switches between them constantly, and everything gets slower. **Prevention:** one shared pool for
CPU work, and libraries that accept the application's pool instead of creating their own.
