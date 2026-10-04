# Audio

How an audio service is built, and how the rest of an application uses it without being tied to it.

Audio has the same overall shape as [rendering](../rendering/README.md): a hardware layer, resources behind
handles, a "world" of proxies, a processing graph, and a narrow public interface. The big difference is
**timing**. A late video frame is a stutter; a late audio buffer is an audible click or dropout. So audio is built
around **hard real-time** rules that rendering can be more relaxed about.

As with rendering, the application never produces samples itself: it **describes** what should be heard (this
sound, from here, this loud, through this bus) and the audio engine turns those descriptions into a continuous
signal.

## The layers inside an audio engine

```
+--------------------------------------------------------------+
| Scene / domain     game objects, timeline clips, UI sounds    |  owned by the app
+--------------------------------------------------------------+
|        commands / sync (what should play, where, how loud)    |
+--------------------------------------------------------------+
| Audio world        emitters, listeners, playing instances,    |  owned by the audio engine
|                    parameters                                 |
+--------------------------------------------------------------+
| Voice management   which instances actually get a voice;      |  limits, priorities,
|                    the rest are virtual                       |  virtualization
+--------------------------------------------------------------+
| Mixing pipeline    voices -> effects -> buses -> master       |  the mixer graph (DSP)
|                    (spatialization, sends, ducking)           |
+--------------------------------------------------------------+
| Audio resources    decoded clips in memory, streamed files,   |  handles, decoding, caching
|                    impulse responses                          |
+--------------------------------------------------------------+
| Device layer       one interface over WASAPI / CoreAudio /    |  platform layer
|                    ALSA / PulseAudio / OpenAL                 |
+--------------------------------------------------------------+
| Sound card / OS mixer                                          |
+--------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [The device layer](01-device-layer.md) | the pull model, the callback, buffer sizes, device changes |
| 2 | [Audio resources](02-audio-resources.md) | clips behind handles; decoded, streamed and compressed; sharing |
| 3 | [The audio world and voices](03-audio-world-and-voices.md) | emitters, listeners, instances, voice limits, virtual voices |
| 4 | [The mixer graph](04-mixer-graph.md) | effects, buses, sends, spatialization, ducking, snapshots |
| 5 | [Threads and timing](05-threads-and-timing.md) | the real-time rules, lock-free queues, latency, sample-accurate timing, ramps |
| 6 | [Sound events](06-sound-events.md) | data-driven sound: what triggers it (code) vs what it sounds like (data) |
| 7 | [Using the audio service](07-using-the-audio-service.md) | the public interface, and how each part of an app uses it |
| 8 | [Coupling: what may depend on audio](08-coupling.md) | safe and unsafe dependencies, null and offline engines |
| 9 | [Assets](09-assets.md) | which assets each layer uses, source vs runtime formats |

## Rendering vs audio, side by side

| | Rendering | Audio |
|---|---|---|
| Hardware layer | RHI over GPU APIs | device layer over OS audio APIs |
| Driven by | the main loop (push: draw a frame) | the device callback (pull: give me samples) |
| Missing a deadline | a dropped frame (a stutter) | a click or dropout (very noticeable) |
| Resources | textures, meshes behind handles | clips (decoded or streamed) behind handles |
| Proxies | sprites, meshes, lights, cameras | emitters, instances, listeners, environments |
| Skipping work | culling | voice limits and virtual voices |
| Processing graph | render graph of passes | mixer graph of effects and buses |
| Designer data | materials, shaders | sound events, mixer snapshots |
| Thread rule | hand over, don't share | hand over, don't share, **and never block** |
| Null version | draws nothing | plays nothing, or mixes offline to a file |

## Patterns in an audio engine

| Pattern | Where |
|---|---|
| Bridge / Adapter | the device layer: one interface, one backend per OS audio API |
| Proxy | clip handles, instance handles |
| Flyweight | one decoded clip shared by every instance playing it |
| Command | main-thread commands queued for the audio thread |
| Producer / consumer | the main thread produces commands, the audio thread consumes them; workers produce streamed audio |
| Lock-free queue | every hand-over to and from the audio thread |
| Pipes and filters | effect chains on voices and buses |
| Strategy | interchangeable effects and spatializers |
| Composite | the bus tree, ending in the master bus |
| State | mixer snapshots ("Paused", "Underwater") |
| Object pool | voices and instance slots, allocated up front |
| Dirty flag | emitters update only when their object moved |
| Null object | an engine that plays nothing, for servers, tools and tests |

## Related

- [Time and scheduling](../time/README.md): clocks, the frame loop, fixed steps and update order that every service runs on.
- [Memory](../memory/README.md): lifetimes, allocators, pools, budgets and GPU memory under every service.
- [Rendering](../rendering/README.md): the same architecture for pictures.
- [Networking](../networking/README.md): the same architecture for sharing state across machines.
- [Input](../input/README.md): the runtime service that carries information in, from devices to intentions.
- [IO](../io/README.md): files, the virtual file system, packages and streaming, under every service's assets.
- [Localization](../localization/README.md): every language and region: keys, formatting, text layout and localized assets.
- [Settings](../04-settings.md): volume sliders and output device choice.
- [Jobs](../06-jobs.md): decoding and streaming on worker threads.
- [Resources](../07-resources.md): the asset system that loads clips and hands them over.
