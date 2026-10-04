# Determinism and replay

A **deterministic** simulation gives exactly the same results from the same starting state and the same inputs.
Networking needs it for lockstep and rollback ([Prediction and timing](../networking/05-prediction-and-timing.md));
replays and reproducible bug reports need it too.

## Rules for a deterministic simulation

- **Advance in fixed ticks** ([The fixed timestep](03-fixed-timestep.md)), never by frame time.
- **Never read real time or the wall clock** inside the simulation. Time inside it is "tick number x step".
- **Seeded random numbers:** a random generator with a known seed, owned by the simulation, advanced only by the
  simulation, in a fixed order.
- **Stable iteration order:** never iterate hash maps or sets whose order may differ between runs or machines; sort,
  or use ordered containers.
- **Inputs per tick** are the only thing entering the simulation from outside ([Input timing](../input/05-timing.md)).
- **No dependence on frame rate, thread timing or memory addresses.**

## Floating point

The same floating-point code can give slightly different results on different CPUs, compilers or optimization
settings. Within one build on one platform, results are usually reproducible; across platforms they are not
guaranteed. Cross-platform lockstep games often use **fixed-point arithmetic** for the simulation for this reason.

## Replay

With a deterministic simulation, a session is fully described by:

```
starting state (or a save) + the inputs for every tick
```

Recording those is tiny compared with recording the state, and replaying them reproduces the session exactly: for
watching replays, for automated tests ("this replay must end with the same state"), and for bug reports.

## Time travel debugging

Combining replay with periodic snapshots of the state (the **Memento** pattern) lets an editor scrub a recorded
session backwards and forwards: jump to the snapshot before the moment of interest, then re-simulate to the exact
tick. Bugs that happen once in an hour of play become reproducible in seconds.

## Checking determinism

Compute a **hash of the simulation state** every tick (or every few ticks) and compare across runs or machines. The
first tick where hashes differ points at where determinism broke.
