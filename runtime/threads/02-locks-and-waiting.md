# Locks and waiting

The primitives that make threads take turns, and wait for each other without burning CPU.

## Mutexes

A **mutex** (mutual exclusion lock) lets one thread at a time into a **critical section**:

```
lock (cache.mutex) {                 // scoped: always released, even on an exception
    entry = cache.find(id);
    if (entry == null) cache.insert(id, placeholder);
}
```

- Always lock in a **scope** (`lock`, `synchronized`, `try/finally`, RAII guards), so an early return or an exception
  can't leave it held.
- Keep critical sections **short**: no file reads, network calls, callbacks into unknown code, or logging while holding
  a lock.
- Uncontended locks are cheap on modern platforms. **Contended** locks are expensive: the waiting thread may be put to
  sleep and woken up later, which costs microseconds.
- Some mutexes are **recursive** (the same thread can lock them twice). They hide design problems and are best avoided.

## Condition variables

A **condition variable** lets a thread sleep until something becomes true ("the queue is not empty"):

```
lock (queue.mutex) {
    while (queue.isEmpty() && !stopping)   // a loop, never an if
        queue.notEmpty.await();             // releases the mutex while sleeping
    if (stopping) return;
    item = queue.pop();
}
```

The **loop** matters: a thread can wake without being signaled (a **spurious wakeup**), or another thread may have taken
the item first. The condition is always re-checked with the lock held. The thread that changes the condition signals
after changing it: `notEmpty.signal()` for one waiter, `signalAll()` when several may proceed.

A lock with its condition variables guarding one object is the **Monitor** pattern.

## Semaphores

A **semaphore** holds a count: `acquire()` takes one (waiting if none are left), `release()` gives one back. Used to
limit concurrency ("at most 4 decoders at once") or to count items in a queue.

## Reader-writer locks

Many readers at once, or one writer alone. Helpful when reads are long and frequent and writes are rare. For short
critical sections they are often **slower** than a plain mutex, and a steady stream of readers can **starve** a writer.
Measure before choosing one; an immutable snapshot is often better ([Handing data between threads](04-handing-data-between-threads.md)).

## Once

Run initialization exactly once, even if several threads ask at the same time (`call_once`, lazy initialization
helpers, static initializers). Hand-written "check, lock, check again" initialization is easy to get wrong without
the right memory ordering ([Atomics and memory ordering](03-atomics-and-memory-ordering.md)).

## Spinning vs sleeping

A **spinlock** loops until the lock is free, without sleeping. It wastes a core while waiting, and if the thread holding
the lock is descheduled, everyone spins for nothing. In application code, spinlocks are rarely right. Most platform
mutexes already **spin briefly** before sleeping, which gives the benefit without the risk.

## Waiting with timeouts

Waits can take a **timeout** ("wait at most 2 seconds for the IO thread to finish"). Timeouts make shutdown robust and
turn silent hangs into reportable errors ([Diagnostics](../05-diagnostics.md)).

## Locks are a last resort

Every lock is a potential deadlock and a point of contention ([Hazards](07-hazards.md)). Prefer, in order:

1. data owned by **one thread** (nothing to lock)
2. **hand-over** through queues, or immutable data ([Handing data between threads](04-handing-data-between-threads.md))
3. a **lock**, hidden inside one small class, never exposed to callers
