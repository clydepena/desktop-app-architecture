# Assets

Time uses little content of its own. Most of what it uses is **configuration**, plus timelines and curves created by
designers.

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Frame loop and pacing** | **pacing presets** | target frame rates per platform, background frame rate, maximum delta |
| **Fixed timestep** | **simulation settings** | step length, maximum steps per frame, interpolation on or off |
| **Time domains** | **tick rates** | simulation and network tick rates, default time scales |
| **Scheduling** | **update schedule** | which systems run in which phase, and their ordering constraints |
| | **time-slicing budgets** | per-frame budgets per category of sliced work |
| **Timelines** | **timeline / sequence assets** | cutscenes, animation sequences, scripted events placed on a timeline |
| | **curve assets** | easing curves, speed ramps, time-scale curves (a slow-motion effect that eases in and out) |

## User settings vs assets vs runtime state

- **User settings:** frame-rate limit, vsync, background frame rate ([Settings](../04-settings.md)). They choose
  between pacing policies; they don't change tick rates.
- **Assets:** tick rates, schedules, budgets, timelines, curves. Saved with the project, the same for everyone.
  Changing a tick rate changes how the simulation behaves, which is a design decision, not a preference.
- **Runtime state:** clocks, accumulators, active timers and coroutines. Game saves store game time and tick counts,
  never the scheduler's internals; timers that matter to gameplay are saved as domain data ("door closes at tick
  N").
