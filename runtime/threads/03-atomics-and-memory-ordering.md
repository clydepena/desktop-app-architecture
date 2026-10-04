# Atomics and memory ordering

The lowest-level tools for sharing data between threads, and the reason "it works on my machine" is not proof.

## Atomic operations

An **atomic** operation happens all at once, as seen by every thread: no other thread observes it half-done.

```
counter.fetchAdd(1)                       // increment without a lock
flag.store(true)                          // publish a value
if (state.compareAndSet(IDLE, RUNNING))   // change only if still the expected value
    ...
```

Without atomics, even `count++` is three steps (read, add, write), and two threads can both read 5 and both write 6.
Atomics are the building blocks of locks, reference counts, lock-free queues and statistics counters.

## Why order matters

Compilers and CPUs **reorder** memory operations to run faster, as long as a single thread can't tell. Another thread
can tell:

```
// thread A                      // thread B
data = loadResult();             if (ready)
ready = true;                        use(data);    // may see ready == true but old data
```

Without the right ordering, thread B can see `ready` become true **before** it sees the new `data`. Neither the
compiler nor the CPU did anything wrong: the program never said the order mattered.

## Acquire and release

The usual fix is a pair of orderings:

- a **release** store ("publish"): everything written before it is visible to whoever sees the store
- an **acquire** load ("subscribe"): everything after it sees what was published

```
// thread A                            // thread B
data = loadResult();                   if (ready.load(ACQUIRE))
ready.store(true, RELEASE);                use(data);           // guaranteed to see the new data
```

Locks include this automatically: unlocking is a release, locking is an acquire. That is why code that uses locks
correctly doesn't need to think about ordering.

## The orderings in common languages

| Ordering | Meaning | C++ | Java | C# |
|---|---|---|---|---|
| relaxed | atomic, but no ordering of other memory | `memory_order_relaxed` | `VarHandle` opaque/plain access | no direct equivalent; `Interlocked` is full fence |
| acquire / release | publish and subscribe | `memory_order_acquire` / `release` | `VarHandle` `getAcquire` / `setRelease` | `Volatile.Read` / `Volatile.Write` |
| sequentially consistent | one global order every thread agrees on | `memory_order_seq_cst` (the default) | `volatile`, `AtomicInteger` | `Interlocked` operations |

Sequential consistency is the easiest to reason about and the default in most languages. Weaker orderings are an
optimization for experts, measured before and after.

## Different hardware, different bugs

x86 processors keep most memory operations in order; ARM processors (phones, Apple Silicon, many servers) reorder much
more. Code with a missing barrier can run correctly for years on desktop x86 and fail on ARM. The compiler can reorder
on any hardware, too. The only safe rule is to state the ordering the program needs, in the language's terms.

## Double-checked locking

The classic example:

```
if (instance == null) {                  // check without the lock
    lock (mutex) {
        if (instance == null)            // check again with it
            instance = new Service();
    }
}
```

Without an acquire/release (or `volatile`) on `instance`, another thread can see a non-null reference to an object
whose fields aren't written yet. Prefer the language's lazy initialization or "once" helper
([Locks and waiting](02-locks-and-waiting.md)).

## Lock-free structures

Structures built only from atomics (lock-free queues, stacks, counters) never block, which is essential where waiting
is not allowed, such as the audio thread ([Threads and timing](../audio/05-threads-and-timing.md)). They are also very hard
to write correctly ([Hazards](07-hazards.md) describes the ABA problem). Use a well-tested library implementation, and
keep the number of different lock-free structures in an application small.
