# Audio resources

The sound data itself: clips, streams, and supporting data such as impulse responses for reverb.

## Clips behind handles

Application code holds a `SoundHandle`, never raw sample arrays. As with textures, the handle (the **Proxy**
pattern) lets the engine load the data later, reload it when the file changes, free it, and use a placeholder
(silence) while it isn't ready.

## Three ways to hold sound data

| Way | How it plays | Good for | Cost |
|---|---|---|---|
| **Decoded in memory** | samples ready, starts instantly | short, frequent sounds: footsteps, clicks, impacts | memory: one second of stereo float at 48 kHz is about 384 KB |
| **Compressed in memory, decoded while playing** | decodes a little ahead of the play position | medium sounds played often | CPU per playing instance |
| **Streamed** | read and decoded from disk in chunks, a little ahead of playback | long sounds: music, ambience, dialogue | a small buffer per instance; disk reads |

Streaming runs on worker threads, which read through the IO scheduler with urgent priority
([The IO scheduler](../io/04-scheduler-and-streaming.md)) and fill a ring buffer that the audio thread reads
from. If the disk is slow, the stream runs dry: streams start a little before they're needed, and their buffers
are sized for the worst disk the app expects.

## Sharing

One decoded clip is shared by every instance playing it (the **Flyweight** pattern). Each **instance** has only its
own state: position in the clip, volume, pitch, loop settings. Ten simultaneous footsteps cost ten small instance
records and one clip.

## Sample-rate conversion

Clips may be recorded at 44.1 kHz, 48 kHz or 96 kHz while the engine mixes at one rate. Converting at load time
costs memory and load time once; converting while playing costs CPU every time. Pitch changes are resampling too,
so engines usually have a good resampler on each voice anyway.

## Loading and freeing

- Decoding happens on worker threads; the finished samples are handed to the engine, never decoded on the audio
  thread.
- A clip still in use by a playing instance is not freed until the instance stops (or is stopped first), the
  audio equivalent of deferred deletion on the GPU.
- Banks ([Sound events](06-sound-events.md)) group clips that load and unload together, e.g. all sounds of one
  level.
