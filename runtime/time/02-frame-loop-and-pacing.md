# The frame loop and pacing

The frame loop is the heartbeat of the application: each iteration measures time, runs the update phases
([5](05-update-phases-and-order.md)), renders and presents. [Lifecycle](../01-lifecycle.md) describes the loop's
place in the application; this page describes its timing.

## Frame time (delta time)

How long the last frame took, measured with the monotonic clock ([Platform clocks](01-platform-clocks.md)).
Everything that changes continuously multiplies by it: "move 5 units per second" becomes "move 5 x delta this
frame".

### Clamp it

After a stall (loading, a debugger breakpoint, the window being dragged, the computer waking from sleep), the next
delta could be seconds long. Unclamped, objects jump through walls and timers fire all at once. A maximum delta
(e.g. 0.1 s) makes the application **slow down** for a moment instead.

### Smooth it, optionally

Averaging delta over a few frames hides measurement jitter, at the cost of reacting slightly late to real changes in
frame rate. Some engines smooth for animation and camera motion while gameplay uses the raw value.

## Frame pacing

An **average** of 60 frames per second can still stutter if frames alternate between 10 ms and 23 ms. Consistent
frame times matter more than the average.

- **Vsync:** wait for the display's refresh before presenting. Steady pacing, no tearing, a little more latency.
- **Frame limiting:** cap the rate (60, or 30 to save battery) with precise waits.
- **Variable refresh rate** displays adjust their refresh to the application, smoothing uneven frames.
- **Present timing:** presenting at the right moment matters as much as rendering fast; it is coordinated with the
  window and display layer.

These are interchangeable **policies** (the **Strategy** pattern), chosen by user settings.

## Background throttling

A minimized, hidden or unfocused application doesn't need 60 frames per second:

- drop to a low rate (a few frames per second), or
- stop the loop and become **event-driven**: sleep until input, a timer or a message arrives

Both save power and keep the computer responsive. Audio and network services keep running on their own threads.

## Patterns

- **Game Loop:** one loop that processes input, advances time, updates and renders, decoupled from how fast the
  hardware runs.
- **Update Method:** each system has an update step called once per frame or tick.

Both are described in Robert Nystrom's *Game Programming Patterns*.
