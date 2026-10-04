# Jobs

Work that must not freeze the UI: loading, importing, building, searching, saving large files, generating
thumbnails.

The threads, locks, queues and task systems underneath are described in
[Threads and synchronization](threads/README.md).

## The shape

```
main thread                      worker threads
-----------                      --------------
start job  ------------------->  run the work (no UI, no GPU)
keep drawing the UI               report progress  --> queued
                                  finish            --> result queued
each frame: run queued results
  (update the document, the UI)
```

- **Workers never touch the UI or the main-thread-only systems** (GPU context, most UI toolkits, documents
  being edited). They compute; the main thread applies.
- **Results come back through a main-thread queue** that the main loop empties every frame ([Lifecycle](01-lifecycle.md)).

```
interface Jobs {
    <T> Job<T> run(String title, Work<T> work);     // background; returns a handle
    void onMainThread(Runnable r);                   // queue for the next frame
    boolean isMainThread();
}
interface Job<T> {
    void cancel();
    double progress();                               // 0..1, or -1 if unknown
    Job<T> then(Consumer<T> onMainThread);           // continuation, on the main thread
    Job<T> onError(Consumer<Throwable> onMainThread);
}
```

## Thread pools

- **CPU work** (decoding, compressing, searching): a pool sized to the number of cores minus one, so the main
  thread keeps a core.
- **Waiting work** (reading files, network): many cheap threads or async I/O; they mostly wait. File reads go
  through the IO scheduler, which adds priorities and cancellation ([IO](io/04-scheduler-and-streaming.md)).
- Worker threads are **daemon/background** threads, so a forgotten job never keeps the app from exiting.

## Progress and cancellation

- Jobs report progress (a fraction, plus a message: "Importing 12/40 textures").
- A job list or status-bar indicator shows running jobs, each with a Cancel button.
- **Cancellation is cooperative:** the work checks `isCancelled()` between steps and stops cleanly. Threads are
  never killed.
- On shutdown ([Lifecycle](01-lifecycle.md)), running jobs are canceled or awaited before services are freed.

## Errors

An exception in a job must not disappear. Catch it in the worker, hand it to the main thread, and report it
([Diagnostics](05-diagnostics.md)) along with the job's title.

## Stale results

A result can arrive after the thing it was for changed or closed: a thumbnail for a deleted file, a load for a
resource that was reloaded again since. Give each request a ticket (a counter or the target's version); when the
result arrives, apply it only if its ticket is still current.

## Logging from workers

Log calls from workers are queued and printed on the main thread, so the log panel (a UI) is only touched there.
