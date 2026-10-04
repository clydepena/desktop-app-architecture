# The audio world and voices

As in rendering, **the audio engine doesn't play the application's objects**. It keeps its own records, the
**audio world**, and mixes only from them.

| Application object | Audio engine record |
|---|---|
| an object that makes sound | an **emitter**: position, velocity, attenuation settings, which bus it plays through |
| a sound being played | a **playing instance**: clip or event, position in it, volume, pitch, loop, state |
| the camera, or the player's head | a **listener**: position, orientation, velocity |
| a reverb zone, a room | an **environment**: shape, effect settings |

## Why proxies

The same reasons as render proxies, with one made stricter:

- **Decoupling:** the engine knows nothing about game objects, timelines or UI widgets.
- **Threading:** the audio thread **must never wait** for the main thread, so it can't read application objects.
  It reads only its own records, updated through queued commands ([Threads and timing](05-threads-and-timing.md)).
- **Consistency:** each mix works from one coherent set of positions and parameters.

## Instance handles

`play()` returns a handle to the playing instance, so the application can stop it, fade it out or change its
parameters later. Like GPU handles, they are an index plus a generation: a handle to a sound that has already
finished is detected as stale instead of controlling whatever sound reused the slot.

One-shot sounds that nobody needs to control (a click, an impact) are started with `playOneShot()` and return
nothing.

## Voice management

Hardware and CPU budgets allow a limited number of sounds to be mixed at once (often 32 to 128 real **voices**),
while a busy scene may request hundreds. Every update, the engine decides which instances get a voice:

- **Priority:** dialogue beats music beats footsteps.
- **Audibility:** the volume after distance attenuation; a sound too quiet to hear doesn't need a voice.
- **Limits per sound:** at most 4 instances of "gunshot" at once; when a fifth starts, the oldest or quietest is
  stopped (or the new one is refused).

### Virtual voices

An instance that loses its voice doesn't stop: it becomes **virtual**. It keeps "playing" silently, its position
in the clip advancing with time, at almost no cost. If it becomes audible again (the listener walks closer), it
gets a voice back and continues from the right point, instead of restarting or staying silent.

Voice management is the audio counterpart of **culling** in rendering: skip the work for what can't be heard.
Voices themselves are pre-allocated (an **object pool**), so the audio thread never allocates.

## Keeping the audio world in sync

As with render proxies: an emitter is created when its object appears, updated when it moves (only if it moved,
using a **dirty flag**), and removed when the object goes away. The listener follows the camera or the player.
These updates are queued as commands and applied by the audio thread at the start of its next buffer.
