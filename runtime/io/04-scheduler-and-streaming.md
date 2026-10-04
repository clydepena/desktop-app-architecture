# The IO scheduler and streaming

The rule shared by every service: **the main thread never waits for storage.** Every read and write is an
asynchronous **request**, and the scheduler decides which run first.

```
io.read("content:/music/theme.ogg", Priority.HIGH)
   .then(bytes -> ...)              // delivered to a worker, or to the main thread
   .onError(error -> ...)
```

## Priorities and deadlines

| Priority | Example | Why |
|---|---|---|
| **urgent** | an audio stream about to run dry | missing it is an audible dropout |
| **high** | textures right in front of the camera | missing it is visible pop-in now |
| **normal** | loading the next area | needed soon |
| **low** | prefetching, thumbnails, cache writes | nice to have |

Requests can also carry a **deadline** ("needed within 200 ms"); the scheduler promotes requests whose deadline is
close.

## Cancellation

The player turned around; the texture isn't needed any more. Canceling a request before it runs saves bandwidth for
what *is* needed. Requests are tied to their owner (an asset handle, a panel, a world chunk), and canceled when the
owner no longer needs them.

## Batching and ordering

Many small reads from one archive are **sorted by offset and merged** into fewer, larger reads. On a hard drive, this
is the difference between seconds and minutes; on an SSD it still reduces overhead.

## Bandwidth budgets

Background work (cache writes, prefetching, thumbnails) is throttled to a share of the available bandwidth, so it
never starves urgent reads. Budgets come from streaming settings ([Assets](09-assets.md)).

## Who does the work

- An **IO thread**, or the operating system's asynchronous IO, issues the reads and writes.
- **Decompression and parsing run on worker threads** ([Jobs](../06-jobs.md)), never on the IO thread (which would
  stop issuing requests) and never on the main thread.
- Results are handed over through queues: the **producer / consumer** pattern with a **priority queue**, the same
  hand-over as every other service.

## Streaming large worlds

Big worlds can't be loaded at once. They are divided into **chunks** (regions, cells), and the scheduler keeps the
right ones loaded:

- chunks **around the player** are requested by distance, with higher priority in the direction of travel and for
  faster movement
- chunks left behind are **unloaded** (after a delay, in case the player turns back)
- textures and meshes stream in **levels of detail**: a small low-detail version first, higher detail as it arrives
- audio streams refill their buffers with **urgent** range reads ([Audio resources](../audio/02-audio-resources.md))

## Missing a deadline

| Symptom | Cause |
|---|---|
| **Pop-in** | low detail (or nothing) shown until higher detail arrives |
| **Hitch** | the app stopped and waited for data it couldn't continue without |
| **Audio dropout** | a stream ran dry |

Pop-in is the designed fallback; hitches mean something waited synchronously, or a request was given too low a
priority.
