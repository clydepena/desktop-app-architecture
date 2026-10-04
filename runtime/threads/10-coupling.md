# Coupling: what may depend on threads

The direction rule: threading is a foundation service. Everything may use it; it depends only on the platform layer
(and on time for timeouts, and memory for queues and stacks). What must be decoupled is not threading itself, but
**knowledge of other systems' threads**.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Any service -> tasks, jobs, `onMainThread`** | the intended doorways |
| **A service -> its own dedicated thread** | the thread is private, reached through the service's own interface |
| **Threading -> platform, time, memory** | layers below or beside it |
| **Code -> thread assertions** | cheap, and they document ownership where it matters |

## Keep decoupled

| System | How it relates instead |
|---|---|
| **Callers of a service** | never know or care which thread does the work; they get futures, events or callbacks with a stated thread |
| **Domain / gameplay code** | runs on the main thread (or in tasks it is given), never creates threads or holds service locks |
| **Locks** | private to one class; never part of a public interface, never held across calls into other systems |
| **Plugins and scripts** | use the task doorway; never start unmanaged threads ([Plugins](../../tool-framework/10-plugins.md)) |
| **Libraries** | accept the application's pool rather than creating their own |

## Mistakes to avoid

- **Blocking the main thread** on files, network, locks held by workers, or `future.get()`.
- **Sharing mutable data** between threads "because it is only read" when anything can write it.
- **Callbacks with an unstated thread**, so receivers touch the UI from a worker.
- **Calling unknown code while holding a lock**: callbacks, events, plugin hooks, logging.
- **A thread per task**, or several libraries each with their own pool: oversubscription.
- **Blocking waits inside pool tasks**, until the pool deadlocks.
- **Hand-written lock-free code** without tests on weakly ordered hardware.
- **Sleeping and polling** instead of waiting on a primitive or a queue.
- **Locks or allocation on the audio thread.**
- **Killing threads** instead of cooperative cancellation.
- **Combining parallel results in completion order** where reproducible results matter.

## Testing the boundary

- **Inline executor:** every service's logic tested with tasks run immediately on the calling thread.
- **Thread assertions** enabled in every test run.
- **Race detection** builds in continuous integration.
- **Stress and shutdown tests** for every queue and every service that owns a thread.
