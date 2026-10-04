# Garbage-collected languages

In Java, C#, JavaScript and similar languages, nobody calls `free`: a **garbage collector** finds objects that are no
longer referenced and reclaims them. The goals of memory management stay the same; the tools change.

## Allocation rate causes stutters

Every short-lived object becomes garbage the collector must eventually process. Many allocations per frame make it run
often, and some collections **pause** the application for milliseconds: visible as stutter.

The equivalent of "steady state allocates nothing":

- **reuse objects**, and **pool** those created and discarded often
- prefer **arrays of primitives** over many small objects (`float[] positions` instead of a list of `Vector` objects)
- avoid **hidden allocations** in hot loops: boxing numbers into objects, iterator objects from `for-each` over some
  collections, lambdas capturing variables, string concatenation, varargs arrays
- keep per-frame temporaries in **reused buffers** owned by the system that needs them

Profilers for these languages show the **allocation rate** per method; the hottest allocators are where to start.

## Native resources need explicit release

GPU textures, audio buffers, file handles, sockets and native memory are invisible to the collector: it only sees a
small object wrapping them, and has no idea it holds 64 MB of video memory. Waiting for the collector (finalizers,
cleaners) is unreliable: it may run late or never.

So native resources are released **explicitly** (`close`, `dispose`), owned by scopes as everywhere else, with
`try-with-resources` (Java) or `using` (C#) for local ones.

## Off-heap memory

Data handed to native code (GPU uploads, audio samples, file contents) lives **outside** the managed heap, in direct
or native buffers:

- the collector never moves or scans it, so native code can use its address safely
- it is often faster to pass to native APIs
- it must be freed explicitly, or it is freed only when the small wrapper object is collected, which may be much later

Large, long-lived binary data (a decoded texture, an audio clip) usually belongs off-heap.

## Leaks still happen

In a garbage-collected language, a leak is **a reference that is no longer needed but still held**:

- listeners and event subscriptions never removed
- caches that never evict
- static collections that only grow
- closures or callbacks that capture large objects
- timers and coroutines still holding their owner

They are found with **heap snapshots** and "what keeps this object alive?" (path to root) analysis. Tying
subscriptions, timers and caches to scopes ([Services and scopes](../02-services-and-scopes.md)) prevents most of them.

## Weak references

A **weak reference** points to an object without keeping it alive. Caches use them (or soft references, which are
cleared only under memory pressure) so cached objects can be reclaimed when nothing else needs them.

## Choosing and tuning the collector

Modern runtimes offer several collectors. For interactive applications, **low-pause** collectors (concurrent,
generational) are the usual choice, trading a little throughput for shorter pauses. Heap size limits matter too: a heap
far larger than needed delays collections and makes each one bigger.
