# Handing data between threads

Most threading bugs come from two threads touching the same data. The safest designs never let them: data moves from
one thread to another, and only one thread uses it at a time.

## Queues

The workhorse: a producer pushes, a consumer pops.

| Queue | Producers / consumers | Typical use |
|---|---|---|
| single-producer single-consumer ring buffer | one / one | main thread to audio thread; lock-free, fixed size, no allocation |
| multi-producer single-consumer | many / one | results from workers to the main thread; log messages |
| multi-producer multi-consumer | many / many | a shared task queue for a worker pool |

Choices to make for each queue:

- **Bounded or unbounded:** a bounded queue has a fixed capacity, so a slow consumer can't use up all memory. When it is
  full, the producer **waits**, **drops** the item, or **replaces** the oldest one, depending on what the data is.
- **Blocking or polled:** a worker blocks until work arrives ([Locks and waiting](02-locks-and-waiting.md)); the main
  thread instead **drains** its queue once per frame ([Lifecycle](../01-lifecycle.md)).
- **Batching:** pushing many small items one at a time costs a synchronization each. Batching them (one push per frame
  or per chunk) costs one.

## Double and triple buffering

Two (or three) copies of a piece of data: one written by the producer, one read by the consumer, swapped at a
synchronization point. The render thread draws frame N from one copy while the main thread builds frame N+1 in the
other ([Frames and threads](../rendering/05-frames-and-threads.md)). The swap is the only moment the threads coordinate.

## Immutable snapshots

Data that never changes after it is published can be read by any number of threads without locks. To "change" it, the
writer builds a **new version** and publishes it with one atomic swap; readers keep using the version they had until
they are done.

```
Settings next = current.with("theme", "dark");   // a new immutable object
settings.store(next, RELEASE);                    // readers see the old or the new one, never half of each
```

This is **copy-on-write**. It suits data read often and changed rarely: settings, key bindings, the list of
installed plugins, a loaded level's static data. The old version is freed once no reader holds it, with reference
counting or deferred destruction ([Ownership and safety](../memory/05-ownership-and-safety.md)).

## Transferring ownership

A buffer filled on a worker is **given** to the main thread, and the worker never touches it again. Nothing is copied
and nothing is locked; only the owner changes. Some languages enforce this (move semantics, ownership types); in
others, it is a convention made visible in names and documentation ("takes ownership of the buffer").

## Messages instead of shared objects

A dedicated thread (audio, IO, network) owns its data and receives **messages**: "play this sound", "read this file",
"send this packet". Other threads never call its objects directly. This is the **Active object** pattern; it turns
locking problems into ordering problems, which are much easier to reason about.

## Futures

A **future** is a single value delivered later: the result of one background job. The caller waits on it, polls it, or
attaches a continuation that runs on a chosen thread ([Task systems](06-task-systems.md)).

## Choosing

| Data | Hand-over |
|---|---|
| a stream of commands or events | a queue |
| the state of a whole frame | double or triple buffering |
| read by many, changed rarely | an immutable snapshot |
| a large result produced once | ownership transfer or a future |
| everything a dedicated thread does | messages |
