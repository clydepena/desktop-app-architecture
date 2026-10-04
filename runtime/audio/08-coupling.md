# Coupling: what may depend on audio

The same direction rule as rendering: **upper layers may depend on the audio service's interface; the audio
engine depends only on what is below it** (the device layer, jobs for streaming and decoding, math, logging
through a queue). Anything that flows upward ("instance finished", "marker reached", "device changed") goes out as
events.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle** | owns the device; suspends and resumes; the single caller of device control |
| **Device layer** | below the engine: the engine depends on it, not the other way round |
| **Resource loaders for clips and banks** | hand decoded data over through the resources doorway |
| **Bridges / audio components** | connecting objects to emitters and instances is their whole purpose |
| **Settings** | volume sliders become bus volumes through the mixer doorway |
| **Editor preview, waveforms, mixer panels, profilers** | the tool layer sits above audio and uses its doorways |

## Keep decoupled

| System | How it reaches audio instead |
|---|---|
| **Domain logic** | calls `play` through bridges or an audio component; never mixes, decodes or touches voices |
| **Undo, documents, persistence** | change domain data (or the mixer document); bridges carry the result to the engine. Undo never stores playing instances |
| **Rendering** | audio and rendering never call each other. Visualizing sound (ranges, meters, waveforms) goes through the editor, which reads audio data and draws with the renderer |
| **Physics** | collisions publish events ("impact, force 12"); a gameplay or bridge system turns them into sounds. Occlusion results are passed in as parameters |
| **Input** | input triggers gameplay, which triggers sounds; the audio engine never reads input |
| **IO** | streams read through the IO layer on worker threads; the audio thread never opens files |

## Mistakes in the wrong direction

- **The audio callback calling into application code,** locking a mutex, allocating or logging: guaranteed
  glitches under load.
- **Domain objects holding raw voices or sample buffers.**
- **The audio engine knowing tool concepts** ("is a preview window open?"): that is a bus the editor uses.
- **Saving playing instances** into documents instead of the data that starts them.
- **Timing visuals with the frame clock** when they must follow the music: read the audio clock instead.

## Testing the boundary

- **A null audio engine** (the **Null Object** pattern): every call accepted, handles handed out, instances finish
  immediately or follow a fake clock. Servers, command-line tools and tests run on it.
- **An offline engine:** the same mixer, writing into a file instead of a device, as fast as the CPU allows. Used
  for exporting audio, and for automated tests that compare the output with a reference recording.

If the domain, documents, undo and the test suite all run on the null engine, the decoupling holds.
