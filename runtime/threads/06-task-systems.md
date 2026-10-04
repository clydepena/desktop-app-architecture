# Task systems

Most code shouldn't create threads or take locks. It should describe **tasks** and let a task system run them on the
worker pool.

## From jobs to tasks

[Jobs](../06-jobs.md) describes long-running background work that the user sees: an import, a search, a save, with
progress and a cancel button. Underneath, the same pool runs many small **tasks**: decode this chunk, cull these
objects, compress this block. Tasks are short (microseconds to milliseconds), numerous, and invisible to the user.

## Parallel loops

The simplest way to use many cores: split a loop into ranges and run them at the same time (**fork-join**).

```
parallelFor(0, particles.count, 1024, range -> {      // 1024 items per task
    for (int i = range.start; i < range.end; i++)
        particles.update(i, dt);
});                                                    // returns when all ranges are done
```

- Each item must be **independent**: no writes to shared data, or each range writes only its own slice.
- The **batch size** matters: too small and the overhead of scheduling dominates; too large and some cores sit idle at
  the end.

## Task graphs

Work with dependencies is described as a **graph**: animation before skinning, skinning before rendering; physics and
audio in parallel. The scheduler starts each task when its inputs are done, and keeps all cores busy. A frame is
often one task graph, rebuilt or reused every frame ([Update phases and order](../time/05-update-phases-and-order.md)).

## Work stealing

Each worker keeps its own queue of tasks. When it runs out, it **steals** from another worker's queue. This balances
load automatically and avoids one shared, contended queue. Most modern task libraries work this way (Java's
fork/join pool, .NET's task scheduler, Intel's TBB and others).

## Waiting inside tasks

A task that blocks a worker (waiting on a lock, a file, or another task) takes a core out of the pool. With enough
blocked tasks, the pool **deadlocks**: every worker waits for a task that no free worker can run. Solutions:

- **continuations:** instead of waiting, a task schedules what comes next to run when its dependency finishes
- a waiting task **helps**: it runs other queued tasks while it waits
- **fibers:** lightweight user-space contexts that can be suspended and resumed on any worker, used by some
  high-performance engines
- slow waiting work (files, network) goes to IO threads or asynchronous APIs, not to the CPU pool

## Futures and continuations

```
Future<Image> image = tasks.run(() -> decode(bytes));
image.then(MAIN_THREAD, img -> thumbnail.set(img));      // continue on a chosen thread
```

A continuation names **where** it runs. Results that touch the UI or documents always continue on the main thread
([Threading models](05-threading-models.md)).

## Async code

Languages with `async`/`await` (C#, JavaScript, Kotlin coroutines, Rust, and virtual threads in recent Java) let
waiting code read like straight-line code, while the runtime suspends and resumes it instead of blocking a thread.
The same questions still apply: which thread does the code resume on, and what does it touch there? Many UI frameworks
resume on the main thread by default; library code often shouldn't.

## Cancellation and errors

Tasks inherit the rules of jobs: cancellation is **cooperative** (checked between steps), and exceptions are captured
and delivered with the result, never lost on a worker ([Jobs](../06-jobs.md)).

## Determinism

Parallel work finishes in a different order every run. When results must be reproducible (simulation, the asset
pipeline, replays), combine results in a **fixed order** (by index, not by completion), and avoid parallel
floating-point reductions whose order changes the result ([Determinism and replay](../time/08-determinism-and-replay.md)).
