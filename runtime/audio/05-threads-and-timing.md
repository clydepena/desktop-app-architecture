# Threads and timing

Audio is the most timing-sensitive part of an application. Its threading rules are stricter than anything else in
the runtime, and break several habits that are fine elsewhere.

## Three kinds of threads

| Thread | Does | Must not |
|---|---|---|
| **Main thread** | decides *what* should play: play, stop, move an emitter, set a parameter | touch mixer state directly |
| **Audio thread** (the device callback) | applies queued commands, voice management results, mixing, effects | block in any way (below) |
| **Worker threads** | decode streamed audio, load and decode clips | be needed "right now" by the audio thread |

## The audio thread's rules

Everything the audio thread does must finish within one buffer's time, every time. So, on the audio thread:

- **No locks** that another thread might hold. Waiting for a lock risks missing the deadline, and a lower-priority
  thread holding it makes things worse (priority inversion).
- **No memory allocation or freeing:** the allocator may take a lock or take a long time.
- **No file IO, no network, no logging to disk, no waiting** on anything.
- **No calls into application code** that might do any of the above.

These rules are why the audio world uses its own records, pre-allocated voices and pools.

## Lock-free queues

Communication happens through **lock-free queues** (single-producer single-consumer ring buffers are the usual
choice):

```
main thread                         audio thread (start of each buffer)
-----------                         -----------------------------------
queue.push(Play(clip, emitter 7))   while (queue.pop(cmd)) apply(cmd);
queue.push(SetBus("Music", -6dB))   ...mix...
                                    events.push(Finished(instance 12))
events.pop(e) -> notify the app     events.push(Levels(master: -12dB))
```

Commands flow down, results flow back (finished instances, markers reached, meter levels). This is the **Command**
pattern and **producer / consumer**, like a render thread, but strictly non-blocking in both directions. Memory
the audio thread no longer needs is handed back the same way, to be freed on another thread.

## Latency

**Latency** is the delay between asking for a sound and hearing it: at least one buffer, usually two or three.

- Smaller buffers: faster response, less margin before a glitch, more CPU overhead per second.
- Larger buffers: safer, but sounds react later to input.

Games usually use 5 to 20 ms. Music tools (instruments, live effects) need much less and often use exclusive
device access.

## Sample-accurate timing

"Start this sound exactly on the beat" can't rely on main-loop timing, which jitters by milliseconds. Commands
carry a **timestamp in samples** (based on the audio clock, not the frame clock), and the audio thread starts the
sound at that exact sample inside the buffer. Rhythm games, sequencers and music synchronization depend on this.

The audio clock is also the right clock to sync visuals to music: the main thread reads the audio position and
places animations accordingly, not the other way round.

## Smoothing parameter changes

Jumping a volume from 1.0 to 0.0 between two samples produces an audible click. The engine **ramps** parameter
changes over a few milliseconds: volume, pan, filter cutoff, pitch. Stopping a sound is a short fade, not a cut.
Rendering has no equivalent of this rule, which makes it a classic audio bug.

## Underruns

When the audio thread misses a deadline, the device plays silence or repeats data: an **underrun**. The engine
counts them and reports them (from the main thread) as the audio equivalent of dropped frames. Frequent underruns
mean too much work per buffer or a rule being broken.

## Related

- [Threads and synchronization](../threads/README.md): atomics, memory ordering, queues and priority inversion in general.
