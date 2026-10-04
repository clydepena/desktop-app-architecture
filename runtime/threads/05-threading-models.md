# Threading models

The single most useful threading decision: write down **which thread owns what**, and check it.

## A typical layout

```
main thread       window messages, input, UI, documents, gameplay updates, applying job results
render thread     builds and submits GPU commands from the main thread's render proxies   (optional)
audio thread      mixes audio in the device callback; never waits, never allocates
IO thread(s)      reads and writes files in priority order
network thread    sends and receives packets
worker pool       everything else: decoding, importing, searching, physics, parallel loops
```

| Thread | Owns | Talks to others through |
|---|---|---|
| main | the UI, documents, the game state, windows | job results queue, render proxies, messages |
| render | the GPU device and command submission | double-buffered proxies |
| audio | voices, the mixer graph | lock-free command and event queues |
| IO | open files, the request queue | requests in, futures out |
| network | sockets, connection state | message queues in both directions |
| workers | nothing permanent: only the task in hand | tasks in, results out |

Smaller applications combine roles (no render thread; IO handled by workers). What matters is that the table
exists, and that every piece of shared state appears in exactly one row.

## The main thread is special

- The operating system requires it for windows and events on several platforms
  ([Platform threads](01-platform-threads.md)).
- Most UI toolkits are single-threaded ([UI toolkit](../ui/README.md)).
- Documents being edited, undo history and selection live there, so commands never race with each other
  ([Undo](../../tool-framework/04-undo.md)).

The main thread must **never block** on slow work. If it waits for a file, a lock held by a worker, or a network
reply, the whole application freezes ([Jobs](../06-jobs.md)).

## Dedicated threads vs the worker pool

| Dedicated thread | Worker pool |
|---|---|
| work that runs all the time, with its own rhythm (audio, render) | work that comes in pieces |
| needs a particular priority or real-time guarantee | normal priority |
| owns long-lived state | owns nothing between tasks |
| one per role | sized to the cores |

Too many dedicated threads compete with the workers for cores. Each one needs a reason.

## Thread checks

Ownership is only real if it is **checked**:

```
void setSelection(Selection s) {
    Threads.assertMainThread();          // fails immediately and loudly in debug builds
    ...
}
```

- Every main-thread-only interface asserts it at its entry points.
- Systems owned by another thread assert that thread ("audio thread only").
- The asserts are cheap enough to keep in development builds, and catch violations at the exact call instead of as a
  rare corruption much later.

## Annotating code

Where the language allows, mark ownership in the code: annotations such as `@MainThread` or `@WorkerThread`, names
like `onMainThread(...)`, or types that can only be used on one thread. Some analyzers check these annotations at
compile time.

## Callbacks state their thread

Every callback and event in a public interface documents **which thread it runs on**: "called on the main thread",
"called on the IO thread; post to the main thread before touching documents". A callback whose thread is unspecified is
a bug waiting to happen ([Events](../03-events.md)).
