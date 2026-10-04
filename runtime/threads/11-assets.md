# Assets

Threading uses no content of its own. What it uses is **configuration**, mostly set per platform.

## Assets by layer

| Layer | Data | What it holds |
|---|---|---|
| **Platform** | **platform profiles** | thread counts, priorities or quality-of-service classes, stack sizes, core reservations per platform (fixed hardware often reserves cores for the OS) |
| **Task systems** | **pool settings** | worker counts (or "cores minus N"), batch sizes for parallel loops, how many cores background work may use |
| **Hand-over** | **queue capacities** | sizes of bounded queues (audio commands, log messages, network packets) and what happens when full |
| **Threading model** | **thread layout** | which optional threads exist (a render thread, a separate network thread), as a build or startup setting |
| **Debugging** | **diagnostic settings** | hang watchdog timeouts, lock-order checking, thread assertions on or off per build |

## User settings

A few choices may be exposed to users, mostly in tools: "use at most N cores for background work" (so an editor stays
responsive while importing or building), or "lower priority while in the background". They are ordinary user settings
([Settings](../04-settings.md)).

## Not assets

Thread names, ownership rules and lock orders are **code**, not data: they are part of the design, checked by
assertions, and documented next to the code that depends on them.
