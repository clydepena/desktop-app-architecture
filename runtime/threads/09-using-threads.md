# Using threads

How the rest of an application uses threading. Most code never creates a thread or takes a lock; it uses these
doorways.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Tasks** | run short work on the pool | `run(work) -> Future`, `parallelFor(range, batch, body)`, `graph()` |
| **Jobs** | long-running, user-visible work | `run(title, work) -> Job` ([Jobs](../06-jobs.md)) |
| **Main thread** | get back to the main thread | `onMainThread(r)`, `isMainThread()`, `assertMainThread()` |
| **Dedicated threads** | start a service's own thread | `start(name, options, body)`, with a message queue |
| **Queues** | hand data between threads | single-producer and multi-producer queues, bounded or not |
| **Primitives** | when nothing above fits | mutex, condition variable, semaphore, atomics, once, thread-local |
| **Platform facts** | size pools and choose priorities | `logicalCores()`, `performanceCores()`, priorities |

## How each part of an application uses it

| Part | How it uses threads |
|---|---|
| **Lifecycle** | starts the pool and dedicated threads in order, and stops them in reverse, waiting for work in flight ([Lifecycle](../01-lifecycle.md)) |
| **Time** | the frame loop on the main thread; timers post to it; sleeping and waiting precision ([Time and scheduling](../time/README.md)) |
| **Memory** | per-thread allocators and arenas; deferred destruction across threads; cache-line padding ([Memory](../memory/README.md)) |
| **Rendering** | an optional render thread fed by double-buffered proxies; command recording on workers ([Frames and threads](../rendering/05-frames-and-threads.md)) |
| **Audio** | a real-time thread, fed only by lock-free queues ([Threads and timing](../audio/05-threads-and-timing.md)) |
| **Networking** | a network thread, or asynchronous sockets, with message queues to the main thread ([Networking](../networking/README.md)) |
| **Input** | events collected on the main thread; high-rate devices may be polled on their own thread and queued ([Input](../input/README.md)) |
| **IO** | IO threads behind the scheduler; decompression on workers ([The IO scheduler](../io/04-scheduler-and-streaming.md)) |
| **Windowing** | windows and their messages on the main thread, as the OS requires ([Windowing and display](../windowing/README.md)) |
| **Scripting** | scripts usually run on the main thread; script runtimes are rarely safe to share between threads ([Scripting](../scripting/README.md)) |
| **UI toolkit** | single-threaded, on the main thread; background results arrive through `onMainThread` ([UI toolkit](../ui/README.md)) |
| **Events** | delivered on the publishing thread, or queued to the main thread; each event documents which ([Events](../03-events.md)) |
| **Diagnostics** | log calls from any thread, queued; thread names in logs, profiler timelines and crash reports ([Diagnostics](../05-diagnostics.md)) |
| **Resources** | loading split into a worker part and a main-thread part ([Resources](../07-resources.md)) |
| **Documents and undo** | main thread only; background work proposes changes, the main thread applies them as commands ([Undo](../../tool-framework/04-undo.md)) |
| **Asset pipeline** | imports and cooks as tasks in dependency order ([The asset pipeline](../../tool-framework/asset-pipeline/README.md)) |
