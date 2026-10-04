# Time and scheduling

How an application keeps time, and decides what runs when.

Time is the service the others **run on**. Every service mentions it: rendering's frames in flight, audio's sample
clock, networking's ticks, input's per-tick snapshots, IO's deadlines. Its job is to answer three questions for
everyone:

- **What time is it?** On *which* clock: real time, game time (which can pause and slow down), simulation ticks,
  the audio clock, the server's clock?
- **How much time passed?** Smoothly and safely, even after a stall.
- **What runs when?** In which order each frame, which steps run at a fixed rate, what is scheduled for later.

It has the familiar shape (a platform layer, abstractions over hardware clocks, a narrow interface, the same
dependency direction). Its special problem is that **time looks simple and isn't**: clocks jump, sleeping is
imprecise, frame times are uneven, floating-point seconds lose precision, and one wrong clock makes a menu freeze
when the game pauses or a game run faster on a faster computer.

## The layers inside a time service

```
+------------------------------------------------------------------+
| Consumers          gameplay, physics, animation, UI, audio sync,   |  owned by the app
|                    networking, tools                               |
+------------------------------------------------------------------+
| Scheduling         update phases and order, timers, delays,        |  what runs when
|                    coroutines, time slicing                         |
+------------------------------------------------------------------+
| Time domains       real time, game time (pause, scale), fixed      |  which clock
|                    ticks, audio clock, network time, timelines      |
+------------------------------------------------------------------+
| Frame loop and     frame time, clamping, smoothing, fixed-step     |  the heartbeat
| pacing             accumulator, vsync, frame limiting               |
+------------------------------------------------------------------+
| Platform clocks    monotonic high-resolution clock, wall clock,     |  platform layer
|                    sleeping, timer resolution                       |
+------------------------------------------------------------------+
| OS, hardware timers                                                   |
+------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform clocks](01-platform-clocks.md) | monotonic vs wall clock, sleeping and its precision |
| 2 | [The frame loop and pacing](02-frame-loop-and-pacing.md) | frame time, clamping, smoothing, pacing, vsync, limiting, background throttling |
| 3 | [The fixed timestep](03-fixed-timestep.md) | the accumulator, interpolation, the spiral of death, what goes where |
| 4 | [Time domains](04-time-domains.md) | several clocks, the clock hierarchy, choosing the right clock, precision |
| 5 | [Update phases and order](05-update-phases-and-order.md) | phases, ordering constraints, one-frame lag, system schedulers |
| 6 | [Timers, delays and coroutines](06-timers-and-coroutines.md) | timers on clocks, ownership and cancellation, coroutines |
| 7 | [Time slicing and budgets](07-time-slicing-and-budgets.md) | spreading work over frames, idle work, scheduling jobs into phases |
| 8 | [Determinism and replay](08-determinism-and-replay.md) | same inputs, same results; recording and replaying ticks |
| 9 | [Using the time service](09-using-the-time-service.md) | the public interface, how each part of an app uses it, editors |
| 10 | [Coupling: what may depend on time](10-coupling.md) | safe and unsafe dependencies, manual clocks, stress pacing |
| 11 | [Assets](11-assets.md) | pacing presets, tick rates, schedules, timelines and curves |

## How time compares with the other services

| | Rendering | Audio | Networking | Input | IO | Time |
|---|---|---|---|---|---|---|
| Role | output | output | out and in | in | storage | **the heartbeat everyone runs on** |
| Platform layer | GPU APIs | audio devices | sockets | OS input | file systems | **OS clocks and timers** |
| Driven by | the main loop | the device callback | messages, ticks | OS events | requests | **itself: it is the loop** |
| Missing a deadline | dropped frame | dropout | lag | missed press | hitch | **stutter, spiral of death** |
| Abstraction | render proxies | emitters, voices | replicas | actions | logical paths | **clocks and time domains** |
| Null version | draws nothing | plays nothing | offline | injected input | in-memory files | **a manual clock** |

## Patterns in a time service

| Pattern | Where |
|---|---|
| Game Loop | the frame loop that drives everything |
| Update Method | systems updated once per frame or tick |
| Fixed timestep with accumulator | fixed simulation steps under a variable frame rate |
| Composite | the clock hierarchy: child clocks pause and scale with their parent |
| Priority queue / timer wheel | timers ordered by when they fire |
| Coroutine | sequential logic spread over frames |
| Topological sort | ordering systems by their declared dependencies |
| Strategy | pacing policies (vsync, limited, unlimited, background) |
| Observer | timer and phase callbacks |
| Test double (fake clock) | a manual clock for tests |

## Related

- [Memory](../memory/README.md): lifetimes, allocators, pools, budgets and GPU memory under every service.
- [Threads and synchronization](../threads/README.md): threads, locks, queues and task systems under every service.
- [Prediction and timing](../networking/05-prediction-and-timing.md), [Threads and timing](../audio/05-threads-and-timing.md),
  [Input timing](../input/05-timing.md), [Frames and threads](../rendering/05-frames-and-threads.md): how each service
  uses time.
- [Windowing and display](../windowing/README.md): windows, displays and scale; hands surfaces to rendering through the application shell.
- [Scripting](../scripting/README.md): behavior added by designers, modders and users, through bindings and sandboxes.
- [Lifecycle](../01-lifecycle.md): the main loop, which the frame loop here describes in detail.
- [Jobs](../06-jobs.md): threads; time adds *when* jobs run.
