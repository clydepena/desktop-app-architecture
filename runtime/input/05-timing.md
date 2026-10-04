# Timing

Input arrives whenever the user acts; the application uses it at fixed points (each frame, or each fixed simulation
tick). Bridging the two without losing or distorting anything takes a few rules.

## Snapshots per frame or tick

Gameplay reads a consistent **input snapshot** for each tick:

- which actions are held, with their values
- which were **pressed** or **released since the last tick**
- accumulated movement (mouse deltas, scroll)

Reading live device state halfway through a tick gives different answers to different systems in the same tick.
A snapshot taken once, at the start, gives everyone the same answer.

## Don't miss presses between ticks

If a key goes down and back up between two fixed ticks, polling "is it down?" at each tick never sees it. The
input layer **latches**:

- "pressed at least once since the last tick" stays true until the tick has read it
- mouse movement and wheel scrolling are **summed** since the last tick, not sampled

This matters most when the simulation runs slower than the display (a 30-tick simulation at 144 frames per
second), or when the app stalls for a moment.

## Frame-rate independence

- **Mouse deltas are distances** already: rotating the camera by "delta x sensitivity" is correct at any frame rate.
  Multiplying them by frame time is a bug: the camera turns slower at high frame rates.
- **Stick values are rates:** "turn this fast" must be multiplied by elapsed time.

Mixing the two up makes camera speed depend on frame rate, one of the most common input bugs.

## Latency

- Read input **as late as possible** before the simulation step that uses it, so it is as fresh as possible.
- **Timestamps** let fast games apply an input at the right moment within a tick (a rhythm game judging a hit to the
  millisecond).
- Long input-to-screen delays usually come from rendering (frames queued ahead) rather than input itself; both
  sides need watching.

## Input as data

The per-tick action snapshot is a small, complete description of what the player wanted. That makes it useful
beyond the local game:

- **Networking:** clients send their action snapshots per tick, not raw key events; servers validate and apply them;
  prediction and rollback replay them
  ([Prediction and timing](../networking/05-prediction-and-timing.md)).
- **Recording and replay:** a recorded sequence of snapshots replays a session exactly, if the simulation is
  deterministic: the **Memento** and **Command** patterns.
