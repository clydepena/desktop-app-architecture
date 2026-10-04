# Assets

Which assets an audio engine uses, and at which layer. Assets are the files created by people (sound designers,
composers, voice actors) or tools, loaded through the asset system ([Resources](../07-resources.md)) and handed to
the engine through its resources doorway.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Device layer** | none (device settings come from settings, not assets) | - |
| **Audio resources** | **audio clips** | sample data: short effects, music, dialogue, ambience |
| | **impulse responses** | recordings of real spaces, used by convolution reverb |
| | **HRTF data sets** | measured head responses, used for 3D sound over headphones |
| **Audio world and voices** | **attenuation presets** | distance curves, minimum and maximum distance, spread |
| | **environment / reverb zone presets** | reverb and filter settings for rooms and areas |
| | **voice and priority settings** | voice budget, per-category priorities and instance limits |
| **Mixer graph** | **mixer layout** | the bus tree, routing, sends, effects on each bus |
| | **snapshots** | named sets of bus and effect settings ("Paused", "Underwater") |
| | **effect presets** | saved settings for one effect (a "small room" reverb, a "radio" EQ) |
| **Sound events** | **event descriptions** | clip lists, randomization, layers, parameters and curves, limits, routing |
| | **banks** | packages of events and clips that load and unload together |
| **Music systems** (optional, above events) | **stems, loops, transition rules, beat and bar markers** | interactive music that changes with the game's state |
| **Dialogue** (optional) | **voice lines, subtitles, localization tables, lip-sync data** | spoken lines per language, with timing for text and animation |

## Source formats and runtime formats

Audio is usually **authored in one format and shipped in another**:

| | Source (what people edit) | Runtime (what the engine loads) |
|---|---|---|
| Effects and short sounds | WAV / AIFF, uncompressed, 48 kHz or higher | uncompressed or lightly compressed (ADPCM), for instant start |
| Music and ambience | WAV, long | compressed for streaming (Ogg Vorbis, Opus, AAC) |
| Dialogue | WAV per line | compressed (Opus is very efficient for speech), per language |
| Events, mixer, snapshots | editable descriptions (JSON, XML, a tool's project) | compact binary descriptions inside banks |

An **import step** converts sources into runtime formats: compression, sample-rate conversion, normalizing
loudness, trimming silence, packing into banks. The settings for each source (compression quality, stream or
decode, loop points) are **import settings**, stored next to the source file (a sidecar), not in it.

## Assets vs settings vs runtime state

- **Assets** describe sounds and how they mix: clips, events, the mixer layout. Created by designers, saved with the
  project, the same for every user.
- **Settings** are the user's choices: master volume, output device, mono output. Saved per user
  ([Settings](../04-settings.md)), and they act on the mixer at runtime without changing the mixer asset.
- **Runtime state** is what is happening now: playing instances, emitter positions, current snapshot. Never saved
  as an asset; game saves store only what is needed to restart it (which music is playing, for example).

## Dependencies

Events reference clips; banks contain events; the mixer layout references effect presets; events reference buses
by name. The asset system loads dependencies first and reports missing ones (an event pointing at a deleted clip)
as problems in the editor ([Diagnostics](../05-diagnostics.md)) rather than failing silently at play time.
