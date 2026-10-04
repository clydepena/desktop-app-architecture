# Time slicing and budgets

Some work is too big to finish within one frame without causing a visible hitch, but doesn't have to be finished in
one frame either. Spreading it over several frames is **time slicing** (also called amortization).

## Budgets per frame

```
sliced(rebuildNavigation, budget = 1 ms per frame)
```

Each frame, the task runs until its budget is used, remembers where it stopped, and continues next frame. Typical
candidates:

- rebuilding pathfinding data a few cells at a time
- checking a tenth of the objects for visibility each frame (each object checked every ten frames)
- AI decisions spread across agents
- incremental garbage collection, cache cleanup, background validation

The task must be written so it can **stop and resume**, and so that partially updated data is still usable (or
double-buffered until the update finishes).

## Idle work

Work with no deadline at all (thumbnail generation, prefetching, indexing a project for search) runs only when the
frame finished early and time is left before the next one. When frames are tight, idle work simply waits.

This is the time counterpart of IO's bandwidth budgets ([The IO scheduler](../io/04-scheduler-and-streaming.md)):
background work never starves what the user is waiting for.

## Scheduling jobs into phases

The jobs system ([Jobs](../06-jobs.md)) decides **where** work runs (which threads). Time adds **when**:

- jobs are started in a phase, with dependencies on other jobs
- the phase that needs their results waits for them (or uses last frame's results, if the design allows a frame of
  delay)
- long jobs that span many frames have no deadline in any single frame, and report progress instead

## Watching the budget

The scheduler records how long each phase, system and sliced task took, and how often budgets were exceeded. The
profiler shows it as a timeline per frame ([Using the time service](09-using-the-time-service.md)).
