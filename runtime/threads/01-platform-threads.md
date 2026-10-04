# Platform threads

The thin wrapper over what the operating system offers: threads, and the facts about the machine they run on.

## Creating threads

The platform layer wraps the OS calls (pthreads on Linux and macOS, the Win32 thread API on Windows, or a language's
own thread class) behind one small interface:

```
interface Threads {
    Thread start(String name, ThreadOptions options, Runnable body);
    void join(Thread t);                        // wait for it to finish
    int  logicalCores();
    int  performanceCores();                    // where the platform reports it
    boolean isMainThread();
}
class ThreadOptions { Priority priority; int stackSize; CoreMask affinity; }
```

Threads are **expensive to create** (a stack, OS bookkeeping) and few should exist. Applications create a small set at
startup (workers, IO, audio, maybe render) and reuse them, instead of a thread per task.

## Names

Every thread gets a **name** ("Main", "Render", "Audio", "Worker 3"). Names appear in debuggers, profilers and crash
reports, and without them a hang report shows a dozen identical, anonymous threads.

## Cores

- **Logical vs physical cores:** with simultaneous multithreading (hyper-threading), one physical core runs two
  logical ones, sharing its execution units. Two threads doing heavy math on one physical core do not run twice as
  fast.
- **Different kinds of cores:** many modern CPUs (most phones, Apple Silicon, recent desktop processors) mix fast
  **performance** cores with slower, power-saving **efficiency** cores. A latency-critical thread that lands on an
  efficiency core runs noticeably slower.
- **Sizing pools:** workers are typically sized from the number of cores, leaving room for the main thread and the
  dedicated ones ([Task systems](06-task-systems.md)).

## Priorities and quality of service

Operating systems let threads ask for more or less urgency:

| Thread | Usual request |
|---|---|
| audio | the highest; on some platforms registered as a real-time or "pro audio" thread |
| main and render | above normal, or "user interactive" |
| workers | normal |
| background work (thumbnails, indexing, uploads) | low, or "background", which may also run on efficiency cores |

Priorities are hints, and their effect differs between platforms. Raising them everywhere helps nothing: when
everything is urgent, nothing is.

## Affinity

Pinning a thread to particular cores is possible, but usually best left to the OS scheduler. It is used on fixed
hardware (consoles), or to keep a critical thread away from busy cores. On general desktops, pinning often makes
things worse.

## Thread-local data

Each thread can have its **own copy** of a variable: a per-thread allocator, a scratch buffer, a random generator, the
"current task" for logging. No locking is needed, because nothing is shared. Thread-local data must be cleaned up when
its thread ends, and doesn't follow work that moves between threads (a task resumed on another worker sees that
worker's copy).

## The operating system's own rules

Some OS facilities only work on particular threads:

- On macOS, the UI (windows, menus, events) must be used from the **main thread**: the one that started the process.
- On Windows, a window's messages are delivered to the thread that **created** it.
- A graphics context (in OpenGL, for example) is current on one thread at a time.
- Many UI toolkits are single-threaded by design.

This is why the **main thread** is special in every application ([Threading models](05-threading-models.md)), and why
the platform layer exposes `isMainThread()`.

## Sleeping and yielding

Sleeping is only as precise as the OS timer allows, often a millisecond or worse; see
[Platform clocks](../time/01-platform-clocks.md). Threads that wait for work should block on a primitive
([Locks and waiting](02-locks-and-waiting.md)), not sleep and poll.
