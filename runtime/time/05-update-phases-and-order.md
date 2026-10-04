# Update phases and order

Each frame runs its work in a defined order of **phases**:

```
poll input -> early update -> fixed steps (physics, simulation) -> update -> late update (cameras follow targets)
-> animation -> prepare render data (sync proxies) -> render -> present
```

Phases give every system a clear moment: input is fresh before the simulation reads it, the simulation is finished
before cameras follow its results, and everything has settled before render data is extracted.

## Ordering constraints

Within a phase, systems still depend on each other. Instead of relying on the order they happened to be registered
in, systems **declare** their constraints:

```
register(movement,   Phase.FIXED)
register(collisions, Phase.FIXED, after(movement))
register(camera,     Phase.LATE,  after(playerController))
```

The scheduler sorts them by these constraints (a **topological sort** of the dependency graph) and reports cycles as
errors. Adding a system then can't silently change the behavior of others.

## One-frame lag

The typical symptom of a wrong order: a camera reads the player's position *before* the player moved this frame, so
the camera is always one frame behind and the player appears to jitter. Similar bugs: an attachment following last
frame's position, a UI label showing last frame's value. Phases and declared ordering fix them at the source.

## System schedulers

Larger engines turn the phase list into a **system scheduler**: it knows which data each system reads and writes, and
runs systems that don't conflict **in parallel** on the jobs system ([Jobs](../06-jobs.md)), while keeping the
declared order where data is shared. Entity-component-system (ECS) architectures usually come with one.

## Rules

- Every per-frame system belongs to exactly one phase.
- Order dependencies are declared, never implied by registration order.
- Phases are fixed for the whole application; new systems join existing phases rather than inventing new ones
  casually.
