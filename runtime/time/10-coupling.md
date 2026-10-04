# Coupling: what may depend on time

Time is one of the lowest services: **almost everyone may depend on its interface**. The time service depends only
on the platform clocks. Even the frame loop doesn't know what its phases contain: systems register into them, and
the scheduler calls them back (inversion of control by registration, not hard-coded calls).

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle** | drives the loop: the single owner |
| **Every service that needs "now", "how long" or "later"** | reads clocks, registers updates, sets timers |
| **Audio and networking** | *provide* clocks (the audio clock, server time) to the time service |
| **Editor play controls, timelines, profilers** | the tool layer, above time |

## Keep decoupled

| System | How it gets time instead |
|---|---|
| **Simulation / gameplay** | receives its step or delta from the scheduler; never reads the wall clock or measures time itself |
| **Rendering** | receives `alpha` and frame data; doesn't run gameplay timing |
| **Saved games** | store game time or ticks for anything that drives gameplay, never wall-clock times |
| **Domain concepts** | the time service knows clocks, not "day and night" or "turns"; those are domain logic built on clocks |

## Mistakes to avoid

- **The wall clock for durations:** clock jumps break timers and timeouts.
- **Frame-rate-dependent movement** (no delta time), or **frame time used inside fixed steps**.
- **An unclamped delta after a stall:** objects teleport.
- **The wrong clock:** frozen menus during a pause, cooldowns running during a pause.
- **Timers and coroutines that outlive their owners.**
- **Update order left to chance:** one-frame lag, behavior that changes when a system is added.
- **Frame limiting with coarse sleeps only:** uneven pacing.
- **32-bit floats for long-running time:** precision loss after hours.
- **Real time or unseeded randomness inside a deterministic simulation.**

## Testing the boundary

- **A manual (fake) clock**, the classic test double: tests set the time and step it explicitly, so a two-second
  timer test runs instantly and the same way every time.
- **Deterministic stepping:** run the simulation for N ticks with recorded inputs and compare the final state (or its
  hash) with a reference.
- **Stress pacing:** inject artificially slow frames and stalls to test clamping, the maximum steps per frame and
  interpolation.
