# Threads and synchronization

Doing several things at once without corrupting data, freezing, or deadlocking.

Every responsive application uses several threads: the main thread runs the UI, and others render, mix audio, read
files, talk to the network and do heavy work in the background. [Jobs](../06-jobs.md) describes the everyday shape
(start work in the background, apply results on the main thread). This series describes what is underneath: the
platform's threads and locks, how data safely crosses between threads, which thread owns what, task systems, and the
bugs that only appear under concurrency.

It has the familiar shape (a platform layer, a small set of primitives, higher-level patterns, a narrow interface, the
same dependency direction). Its special problems are **correctness and contention**:

- **Correctness:** two threads touching the same data without coordination produce results that are wrong only
  sometimes, on some machines, which makes them the hardest bugs to find.
- **Contention:** coordination costs time. Threads waiting on each other can make a parallel program slower than
  a serial one.

The best concurrent code shares as little as possible: most data has **one owner thread**, and the rest is handed over
through a few well-tested channels.

## The layers inside threading

```
+--------------------------------------------------------------------+
| Consumers          every service: rendering, audio, IO, networking,   |  owned by the app
|                    resources, the asset pipeline, gameplay             |
+--------------------------------------------------------------------+
| Threading model    which thread owns what: main, render, audio, IO,   |  the rules
|                    workers; thread checks                              |
+--------------------------------------------------------------------+
| Task systems       jobs, task graphs, parallel loops, work stealing,  |  most code lives here
|                    futures and continuations                           |
+--------------------------------------------------------------------+
| Hand-over          queues, double buffers, snapshots, messages        |  data crossing threads
+--------------------------------------------------------------------+
| Primitives         mutexes, condition variables, semaphores, atomics, |  the building blocks
|                    once, thread-local data                             |
+--------------------------------------------------------------------+
| Platform           threads, priorities, affinity, names, core counts  |  platform layer
+--------------------------------------------------------------------+
| OS scheduler, CPU cores and caches                                     |
+--------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform threads](01-platform-threads.md) | creating threads, names, priorities, cores, thread-local data, OS main-thread rules |
| 2 | [Locks and waiting](02-locks-and-waiting.md) | mutexes, condition variables, semaphores, reader-writer locks, spinning |
| 3 | [Atomics and memory ordering](03-atomics-and-memory-ordering.md) | atomic operations, why ordering matters, acquire and release |
| 4 | [Handing data between threads](04-handing-data-between-threads.md) | queues, double buffering, immutable snapshots, ownership transfer |
| 5 | [Threading models](05-threading-models.md) | which thread owns what, thread checks, dedicated threads vs workers |
| 6 | [Task systems](06-task-systems.md) | task graphs, parallel loops, work stealing, futures, async code |
| 7 | [Hazards](07-hazards.md) | data races, deadlocks, priority inversion, false sharing, starvation, ABA |
| 8 | [Debugging and testing](08-debugging-and-testing.md) | sanitizers, profiler timelines, stress tests, deterministic schedules |
| 9 | [Using threads](09-using-threads.md) | the public interface, and how each part of an app uses it |
| 10 | [Coupling: what may depend on threads](10-coupling.md) | safe and unsafe dependencies, thread assertions, tests |
| 11 | [Assets](11-assets.md) | thread counts, priorities, core reservations per platform |

## How threads compare

| | Memory | Time | Threads |
|---|---|---|---|
| Role | where everything lives | the heartbeat | **doing several things at once** |
| Platform layer | virtual memory | OS clocks | **OS threads and synchronization** |
| What goes wrong | crashes, leaks, stutter | stutter | **wrong results sometimes, deadlocks, contention** |
| Abstraction | allocators, handles | clocks | **tasks, queues, owner threads** |
| Skipping work | reuse | budgets | **not sharing: one owner, hand-over instead of locks** |
| Test version | tracking allocators | manual clock | **inline (single-threaded) executor, race detectors** |

## Patterns in threading

| Pattern | Where |
|---|---|
| Producer / consumer | queues between every pair of threads |
| Thread pool | workers reused for many short tasks |
| Active object | a dedicated thread with a message queue (audio, IO, render) |
| Future / promise | results of background work |
| Monitor | a lock plus condition variables guarding one object |
| Double buffering | one side written while the other is read |
| Immutable snapshot / copy-on-write | readers never blocked by writers |
| Fork-join | parallel loops |
| Work stealing | balancing tasks across workers |
| Thread confinement | data that only one thread ever touches |
| RAII / scoped lock | locks that can't be left held |
| Null object | the inline executor that runs everything on the calling thread |

## Related

- [Time and scheduling](../time/README.md): the frame loop, sleeping and timers that threads wait on.
- [Memory](../memory/README.md): per-thread allocators, false sharing, deferred destruction across threads.
- [Threads and timing](../audio/05-threads-and-timing.md): the strictest thread in an application.
- [Jobs](../06-jobs.md): the everyday interface to background work.
