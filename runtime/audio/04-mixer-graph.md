# The mixer graph

Every voice runs through a chain of processing (DSP, digital signal processing), then into **buses**, which are
mixed together into the final output:

```
voice -> pitch/resample -> filter -> spatializer ---> SFX bus ------+
voice -> ... ---------------------------------------> Music bus ----+--> Master bus -> limiter -> device
voice -> ... -----------------> Dialogue bus --(send)--> Reverb -----+
                                     |
                                     +--(sidechain)--> ducks the Music bus while someone speaks
```

## Buses

A **bus** (or group) mixes everything routed into it, applies its own volume and effects, and passes the result on.

- Typical buses: SFX, Music, Dialogue, Ambience, UI, all ending in **Master**.
- They form a tree (the **Composite** pattern): turning down "SFX" turns down everything under it.
- User settings act on buses: the "Music volume" slider is the Music bus's volume.
- Buses are where **metering** happens: their levels feed volume meters in a mixer panel.

## Effects

Effects are interchangeable processing units (the **Strategy** pattern) chained in order (**pipes and
filters**): filters (low-pass, high-pass), EQ, compressor, limiter, distortion, delay, reverb, chorus.

- On a **voice**: affects one sound (a muffled radio voice).
- On a **bus**: affects a whole group (all music slightly compressed).
- The **master** usually ends with a **limiter**, so many loud sounds at once never clip.

## Sends

A **send** routes a copy of a signal, at its own level, to another bus, typically a shared effect bus: one reverb
used by every sound in a cave, instead of one reverb per sound. How much each sound sends decides how "wet" it
sounds.

## Spatialization

Turning an emitter's position relative to the listener into what the listener hears:

- **Panning:** left/right (or around a surround layout) from the direction.
- **Distance attenuation:** quieter with distance, following a curve (linear, logarithmic, custom) between a
  minimum and maximum distance.
- **Doppler:** pitch shifts with relative speed.
- **HRTF** (head-related transfer functions): convincing 3D, including above and behind, over headphones.
- **Occlusion and obstruction:** sounds behind walls are quieter and muffled; found by ray casts in the physics
  or game world, then passed in as a parameter.

## Ducking and sidechains

One bus can lower another automatically: music dips while dialogue plays, then comes back. The dialogue bus's
level drives a compressor on the music bus (a **sidechain**).

## Snapshots

A **snapshot** is a named set of mixer settings: "Paused" lowers and muffles everything except the UI,
"Underwater" adds a low-pass filter to SFX, "Cutscene" lowers ambience. Switching snapshots crossfades between them
over a moment. This is the **State** pattern applied to the mix, and it keeps "how the game sounds in this
situation" out of code.

## The graph is data

Like a render graph, the mixer layout (buses, routing, effects, snapshots) is described as nodes and connections
in a file. Sound designers edit it in a mixer panel; the engine builds the processing order from it. Code only
refers to buses and snapshots by name.
