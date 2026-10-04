# Using the audio service

How the rest of an application gets anything heard. As with rendering, the application never produces samples: it
**describes** what should play through a few doorways. The audio engine never calls back into the application;
anything it reports ("instance finished", "marker reached", "device changed") goes out as events, delivered on the
main thread.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Playback** | start sounds and events, get instance handles | `play(sound, emitter) -> InstanceHandle`, `playOneShot(sound)` |
| **Instances** | control what's playing | `stop(h, fade)`, `setVolume(h, v)`, `setPitch(h, p)`, `setParameter(h, "rpm", x)` |
| **Audio world** | emitters, listeners, environments | `createEmitter() -> id`, `setPosition(id, p)`, `setListener(transform)` |
| **Mixer** | buses and snapshots | `setBusVolume("Music", dB)`, `applySnapshot("Paused")` |
| **Resources** | hand decoded or streamable data to the engine | `createClip(samples)`, `openStream(source)`, `loadBank(bank)` |
| **Analysis / readback** | levels, spectrum, playback position | `meter("Master")`, `position(h)` (values reported from the audio thread) |
| **Effects** (extension point) | add custom processing | `registerEffect("bitcrusher", factory)` |
| **Device and lifecycle** | open and close the device, suspend everything | `init(deviceSettings)`, `suspend()`, `resume()` |

## How each part of an application uses it

### The lifecycle owns the device

The main loop ([Lifecycle](../01-lifecycle.md)) opens the device at startup, suspends audio when the app is
minimized or loses focus (if the settings say so), and at shutdown stops all instances and closes the device. It is
the only caller of device control.

### Resources hand over decoded data

Short clips are decoded on worker threads by the asset system ([Resources](../07-resources.md)) and handed over;
long files become streams that the engine's own workers read. Banks load through the same path.

### The domain goes through bridges

As with rendering, domain objects don't know the audio engine. A small **bridge** connects them: an "audio source"
component that owns an emitter, copies its object's position when it moves, and calls `play` on gameplay events.
The camera's (or player's) bridge updates the listener.

```
object appears       -> bridge: emitter = audio.createEmitter()
object moved         -> bridge: audio.setPosition(emitter, ...)   (only if dirty)
"footstep" happened  -> bridge: audio.play("footstep", emitter, params)
object removed       -> bridge: stop its instances, destroy the emitter
```

Game code says "play footstep here", never "fill this buffer".

### The UI

UI sounds (clicks, notifications) play as one-shots on a **UI bus**, with no position, so they ignore the
listener and are unaffected by in-game snapshots like "Underwater".

### Settings

Master, music, SFX and dialogue volume sliders map to bus volumes. The output device choice, mono output (an
accessibility option) and "mute when in background" are settings the engine reads itself
([Settings](../04-settings.md)).

### Editors

- **Preview / audition:** play a clip or event on a separate "editor preview" bus, even while the scene is paused
  or not running.
- **Waveform and spectrum views:** decoded samples are turned into an image on a worker thread and cached, like
  thumbnails.
- **Live mixer editing:** moving a bus fader in a panel changes the sound immediately. Mixer layouts are usually
  documents, so the change is an undoable edit like any other.
- **Visualizing 3D sound:** attenuation ranges drawn as circles or spheres around emitters. That is **immediate
  drawing in the renderer**, fed by the audio bridge's data, not something the audio engine draws.
- **Profiling:** voice counts, virtual voices, CPU per effect, underruns, stream buffer health.

### Diagnostics

The engine publishes statistics and underrun counts. It never logs from the audio thread: messages are queued and
written by the main thread ([Diagnostics](../05-diagnostics.md)).

## The same shape as rendering

Doorways, proxies, bridges and events: the structure matches [Using the renderer](../rendering/06-using-the-renderer.md)
almost line for line. The differences are the device's pull model and the audio thread's rule of never blocking.
