# IO

How storage is made simple, uniform and safe for the rest of an application.

IO is the service the others **stand on**: rendering, audio and input loaders read through it, persistence and
settings write through it, networking downloads into it. Its job is to give one way to name files and one way to
read and write them, wherever they actually live (a folder, a packed archive, a network share, memory), on any
operating system, without ever freezing the application.

It has the familiar shape (a platform layer, an abstraction that hides where data lives, a scheduler, a narrow
interface, the same dependency direction). Its special problems are **slowness and variety**: storage is thousands
of times slower than memory, behaves differently on SSDs, hard drives, network drives, consoles and phones, and
fails in many ways (missing, locked, read-only, disk full, unplugged).

## Where IO stops

Two other documents sit directly above IO:

- **[Persistence](../../tool-framework/03-persistence.md)** is about *documents*: their formats, versions, autosave
  and recovery.
- **[Resources](../07-resources.md)** is about *loaded assets*: handles, caches, placeholders.

**IO is the layer under both:** paths, files, streams, packages and scheduling. It knows nothing about documents or
assets.

## The layers inside an IO service

```
+-----------------------------------------------------------------+
| Consumers          resources, persistence, settings, logging,     |  owned by the app
|                    audio/texture streaming, downloads, tools      |
+-----------------------------------------------------------------+
| Formats            readers/writers for text and binary,           |  shared building blocks
|                    compression, checksums, encryption             |
+-----------------------------------------------------------------+
| Streams            sequential reading/writing, decorators         |  stream pipeline
|                    (buffering, decompressing, hashing)            |
+-----------------------------------------------------------------+
| IO scheduler       async requests, priorities, deadlines,         |  never block the main thread
|                    batching, cancellation, bandwidth budgets      |
+-----------------------------------------------------------------+
| Virtual file       logical paths, mount points, search order,     |  hides where data lives
| system (VFS)       folders / archives / memory / network backends |
+-----------------------------------------------------------------+
| Platform           files, directories, memory mapping, file       |  platform layer
|                    watching, permissions, well-known folders       |
+-----------------------------------------------------------------+
| OS file systems, storage hardware                                   |
+-----------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform and file systems](01-platform-and-file-systems.md) | files, memory mapping, watching, well-known folders, platform differences |
| 2 | [The virtual file system](02-virtual-file-system.md) | logical paths, mounts, backends, overlays and search order |
| 3 | [Packages](03-packages.md) | archives: why, layout, compression, patching, integrity, chunks |
| 4 | [The IO scheduler and streaming](04-scheduler-and-streaming.md) | async requests, priorities, cancellation, batching, budgets, streaming worlds |
| 5 | [Streams and formats](05-streams-and-formats.md) | stream decorators, ranges, binary and text readers, compression, hashes |
| 6 | [Writing safely](06-writing-safely.md) | atomic writes, flushing, failures, cloud-synced folders, locking |
| 7 | [Using the IO service](07-using-the-io-service.md) | the public interface, and how each part of an app uses it |
| 8 | [Coupling: what may depend on IO](08-coupling.md) | safe and unsafe dependencies, in-memory and failing backends |
| 9 | [Assets](09-assets.md) | mount tables, package manifests, file ordering, streaming settings |

## How IO compares with the other services

| | Rendering | Audio | Networking | Input | IO |
|---|---|---|---|---|---|
| Information flows | out | out | out and in | in | **to and from storage** |
| Platform layer | GPU APIs | audio devices | sockets | OS input | **file systems** |
| Driven by | the main loop | the device callback | messages and ticks | OS events | **requests, by priority** |
| Missing a deadline | dropped frame | click or dropout | lag, desync | missed press | **hitch or pop-in** |
| Abstraction over hardware | render proxies, handles | emitters, voices | replicas, remote proxies | actions | **logical paths over mounts** |
| Skipping work | culling | virtual voices | relevancy | consumption | **caching, canceling, prefetching** |
| Null version | draws nothing | plays nothing | offline, loopback | injected input | **in-memory file system** |

## Patterns in an IO service

| Pattern | Where |
|---|---|
| Bridge / Adapter | the platform layer over each OS's file APIs |
| Facade | the VFS: one front for every kind of storage |
| Proxy | a logical path stands in for a physical location; file handles |
| Strategy | interchangeable backends: folder, archive, memory, network |
| Chain of responsibility / layered composite | overlays: mounts searched in order, the first match wins |
| Decorator | streams wrapping streams: buffering, decompression, hashing, encryption |
| Priority queue + producer / consumer | the IO scheduler and its worker hand-over |
| Future / promise | asynchronous reads and writes |
| Observer | file watching |
| Null object / fake | the in-memory backend for tests; failing backends for fault injection |

## Related

- [Time and scheduling](../time/README.md): clocks, the frame loop, fixed steps and update order that every service runs on.
- [Memory](../memory/README.md): lifetimes, allocators, pools, budgets and GPU memory under every service.
- [Threads and synchronization](../threads/README.md): threads, locks, queues and task systems under every service.
- [Audio resources](../audio/02-audio-resources.md): streaming audio through the scheduler.
- [Input](../input/README.md), [Networking](../networking/README.md), [Rendering](../rendering/README.md),
  [Audio](../audio/README.md): the other runtime services.
- [Localization](../localization/README.md): every language and region: keys, formatting, text layout and localized assets.
- [Jobs](../06-jobs.md): decompression and parsing on worker threads.
- [Resources](../07-resources.md), [Persistence](../../tool-framework/03-persistence.md): the layers directly above IO.
- [The asset pipeline](../../tool-framework/asset-pipeline/README.md): produces the cooked data and package manifests IO reads.
