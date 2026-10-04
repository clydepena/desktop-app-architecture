# Sound events

Mature audio systems (FMOD, Wwise, and the audio systems built into large engines) add one more layer above
clips: **sound events**, authored by sound designers.

```
event "footstep"  = one of 6 clips at random, pitch +-5%, clips chosen by a "surface" parameter, max 4 at once
event "engine"    = layered loops crossfaded by an "rpm" parameter
event "ambience"  = a base loop, plus random bird calls every 3-10 seconds
```

The code only says:

```
audio.play("footstep", emitter, params("surface", "gravel"));
engineInstance.setParameter("rpm", 3200);
```

and designers change the sound completely, without touching code.

## What it separates

- **What triggers a sound:** code (gameplay, UI, a timeline). It names events and sets parameters.
- **What it sounds like:** data (clips, randomization, layering, effects, routing). Designers own it.

The same separation as materials and shaders for visuals: code says "draw with this material", artists decide what
the material looks like.

## What an event can contain

- **Clip selection:** random from a list (without repeating the last one), sequential, or chosen by a parameter.
- **Randomization:** volume and pitch ranges, so repeated sounds don't feel mechanical.
- **Layers:** several sounds played together, each with its own volume curve.
- **Parameters:** named values (`rpm`, `surface`, `health`) that drive volumes, crossfades, effects and clip
  choice through curves.
- **Timelines and markers:** sections, loops, and markers the game can wait for ("the bell rings now").
- **Instance limits and priority**, used by voice management.
- **Routing:** which bus, which sends, which spatialization settings.

## Banks

Events and their clips are packed into **banks**: files that load and unload together (all sounds of one level,
all UI sounds). Loading a bank makes its events playable; unloading frees their clips once nothing is playing them.

## Patterns

- **Data-driven design:** behavior lives in data files, not code.
- **Flyweight:** one event description, many playing instances.
- **Interpreter:** the engine reads the event description and executes it at runtime.
