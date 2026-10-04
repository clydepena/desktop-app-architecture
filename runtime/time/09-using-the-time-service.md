# Using the time service

How the rest of an application uses time: by reading the right clock, registering into phases, and scheduling work
for later. The time service never calls application code directly; systems **register** updates, timers and
coroutines, and the scheduler runs them (inversion of control by registration).

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Clocks** | read the time on the right clock | `realTime()`, `gameTime()`, `deltaTime(clock)`, `tick()`, `alpha()` |
| **Control** | pause and scale | `pause(clock)`, `resume(clock)`, `setTimeScale(clock, 0.25)`, `step(clock)` |
| **Clock hierarchy** | local clocks | `createClock(parent) -> Clock` (a slow-motion bubble, one character's time) |
| **Update registration** | run each frame or tick | `register(system, Phase.FIXED, after(physics))` |
| **Timers** | run later | `after(2.0, clock, callback) -> TimerHandle`, `every(0.5, clock, callback)`, `cancel(handle)` |
| **Coroutines** | sequential logic over time | `start(owner, routine)`, `waitSeconds(2, clock)`, `waitUntil(condition)`, `waitTicks(10)` |
| **Budgets** | spread work over frames | `sliced(task, budgetPerFrame)`, `whenIdle(task)` |
| **Pacing** | frame rate policy | `setTargetFrameRate(60)`, `setVsync(on)`, `setBackgroundFrameRate(5)` |
| **Statistics** | frame timing | frame-time history, per-phase and per-system timings, budget overruns |

## How each part of an application uses it

| Part | How it uses time |
|---|---|
| **Lifecycle** | runs the frame loop: measures frame time, runs the phases, runs fixed steps with the accumulator, applies pacing. The time service's only driver ([Lifecycle](../01-lifecycle.md)) |
| **Gameplay** | reads game time and its delta, registers into phases, uses game-time timers and coroutines; never touches real time or the wall clock |
| **Physics and networked simulation** | run in the fixed-step phase and count ticks |
| **Rendering** | uses `alpha` to interpolate between simulation states; pacing and vsync are applied by the loop ([Frames and threads](../rendering/05-frames-and-threads.md)) |
| **Audio** | keeps its own sample clock, exposed as a time domain; music-synchronized gameplay reads it |
| **Networking** | maintains the synced server time and the shared tick number, exposed as clocks |
| **Input** | latches events between ticks and builds a snapshot per tick at the start of the fixed-step phase ([Input timing](../input/05-timing.md)) |
| **UI** | animates with real time, so menus keep working while the game is paused |
| **IO and jobs** | use real-time deadlines and budgets; background work runs in idle time |
| **Diagnostics** | frame-time graphs, timings per phase and system ([Diagnostics](../05-diagnostics.md)) |
| **Settings** | frame-rate limit, vsync and background rate are user settings ([Settings](../04-settings.md)); tick rates are asset data, not user settings |

## Editors and tools

- **Pause, step and slow motion:** pause the game, advance exactly one frame or one tick, run at 10% speed. These
  control game time and leave the editor's own real-time UI running.
- **Play time is separate from editor time:** the editor keeps animating, accepting input and drawing while the game
  is paused or not running at all.
- **Timelines and sequencers:** scrub a cutscene back and forth on timeline time, with everything evaluated from the
  time ([Time domains](04-time-domains.md)).
- **Profiler timelines:** frame-time graphs, phase and system timings per frame, spikes highlighted.
- **Time travel debugging**, where the simulation is deterministic ([Determinism and replay](08-determinism-and-replay.md)).
