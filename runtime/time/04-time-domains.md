# Time domains

One "current time" isn't enough. Applications keep several clocks, each answering a different question.

| Clock | Pauses? | Scales? | Used by |
|---|---|---|---|
| **Real (unscaled) time** | no | no | UI animations, menus, network timeouts, profiling |
| **Game time** | yes | yes (slow motion, fast forward) | gameplay, game animations, cooldowns |
| **Simulation ticks** | with game time | in whole steps | physics, networked and deterministic logic; counts ticks, not seconds |
| **Audio clock** | follows playback | n/a | music synchronization: the samples actually played are the truth |
| **Network / server time** | no | no | timestamps shared with other machines |
| **Timeline / media time** | yes, and can **scrub** backwards | yes | cutscenes, animation sequences, video, an editor's timeline |

Some clocks are **provided** by other services: the audio engine reports its playback clock
([Threads and timing](../audio/05-threads-and-timing.md)), the network service maintains the synced server time
([Prediction and timing](../networking/05-prediction-and-timing.md)). The time service exposes them alongside its own.

## The clock hierarchy

Clocks form a tree (the **Composite** pattern):

```
real time
  +-- game time                       (paused by the pause menu, scaled by slow motion)
        +-- slow-motion bubble        (a region where time runs at 25%)
        +-- character "Hero"          (a power-up that speeds one character up)
  +-- UI time                         (never paused)
```

Pausing or scaling a parent affects every child. Each consumer picks the clock that matches what it represents.

## Choosing the wrong clock

| Mistake | Result |
|---|---|
| a pause menu animated with **game** time | the menu freezes, because game time is paused |
| a cooldown measured in **real** time | it keeps counting while the game is paused |
| music-synchronized effects timed with the **frame** clock | they drift, because the audio clock is the truth |
| gameplay timestamps from the **wall** clock | jumps, and different results on every machine |

## Precision

A 32-bit floating-point number has 24 bits of precision, so the larger the value, the coarser its steps. A clock
stored as float seconds can only change in steps of about 1 ms once it passes 10,000 s (under 3 hours), and about
8 ms past 100,000 s (about a day): coarser than a frame at 144 Hz. Frame deltas computed from it become uneven and
motion stutters. Use 64-bit values, integer microseconds or nanoseconds, or **tick counts**, and convert to float only
for short local differences.

## Timeline time

Timeline time can jump and move backwards (scrubbing in an editor, skipping a cutscene). Anything driven by it must be
**evaluated from the time** ("where is everything at 3.2 s?") rather than accumulated frame by frame ("move a little
further each frame"), or scrubbing back leaves things in the wrong place.
