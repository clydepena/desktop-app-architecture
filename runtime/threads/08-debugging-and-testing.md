# Debugging and testing

Concurrency bugs hide from ordinary tests: they need a particular timing, which may happen once in a million runs, or
only on a customer's machine. Finding them takes tools built for the purpose.

## Race detectors

- **ThreadSanitizer** (in Clang and GCC) instruments memory accesses and reports data races with both stack traces,
  even when the race didn't cause a visible failure on that run. Other tools (Valgrind's Helgrind, and commercial
  analyzers) do similar work.
- Managed languages have their own: concurrency stress harnesses such as Java's jcstress, and analyzers for
  common mistakes in .NET.
- Run race detection regularly in automated builds; a race it finds today is a crash report avoided next month.

## Lock checking

Debug builds can record the order in which every thread takes locks, and report a possible deadlock the first time two
locks are taken in conflicting orders, without the deadlock ever happening. Some platforms and libraries provide this;
it is also simple to add to a project's own mutex wrapper.

## Profiler timelines

A profiler that shows **every thread on a timeline** answers most performance questions: which threads are busy,
which are waiting and on what, whether work is spread across cores or serialized behind one lock. Named threads
([Platform threads](01-platform-threads.md)) and named tasks make these timelines readable
([Diagnostics](../05-diagnostics.md)).

## Hangs

When the application freezes, a dump of **all threads' stacks** shows who waits for whom. A watchdog that notices the
main thread hasn't finished a frame for several seconds can capture this automatically and attach it to a crash or
hang report.

## Testing

| Test | What it finds |
|---|---|
| **inline executor** | runs every task immediately on the calling thread; logic tests become deterministic and simple |
| **stress tests** | many threads hammering one structure (a queue, a cache) for a long time, checking invariants after |
| **randomized scheduling** | test harnesses that add random delays or yields at synchronization points, shaking out timing bugs |
| **shutdown tests** | starting and stopping services repeatedly, with work in flight; finds leaks, hangs and use after free |
| **thread assertions** | turned on in all tests, so ownership violations fail immediately |
| **ARM test machines** | code with weak memory ordering bugs that x86 hides ([Atomics and memory ordering](03-atomics-and-memory-ordering.md)) |

## Reproducing

A concurrency bug that can't be reproduced can still be **reasoned about**: list every thread that touches the data,
and what orders their accesses. If the answer is "nothing", the bug is found, whether or not it has been seen yet.
