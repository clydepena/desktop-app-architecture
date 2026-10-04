# The fixed timestep

Some systems must advance in **fixed steps**: physics (stable only with a constant step), networked simulation
(every machine must step identically), deterministic gameplay. Rendering, meanwhile, runs as fast as it can. The
standard way to combine them is the **accumulator**, known from the article "Fix Your Timestep!":

```
accumulator += frameTime                       (clamped)
while (accumulator >= step):                   e.g. step = 1/60 s
    previousState = currentState
    simulate(step)                             fixed, deterministic steps
    accumulator -= step
alpha = accumulator / step                     how far between the last two steps (0..1)
render(interpolate(previousState, currentState, alpha))
```

## Interpolation

The simulation advances in steps, but frames fall between steps. Rendering blends the last two simulation states
using `alpha`, so motion is smooth when the frame rate and the tick rate differ (a 144 Hz display showing a 60 Hz
simulation). The cost: what is drawn is up to one step behind the simulation.

The alternative, **extrapolation** (predicting ahead from the latest state), has no delay but shows wrong positions
whenever motion changes.

## The spiral of death

If one simulation step takes longer to compute than `step` itself, the accumulator grows every frame, the loop runs
more steps to catch up, each frame takes longer, and the application freezes. A **maximum number of steps per frame**
prevents this: beyond it, the remaining time is dropped and the simulation runs slower than real time for a moment.

## Choosing the step

- **Physics:** 50 to 120 Hz is common; fast objects and stiff constraints need smaller steps.
- **Networked games:** the tick rate trades responsiveness for bandwidth and server cost (20 to 128 Hz).
- **Turn-based or slow simulations** can use much larger steps.

## What goes where

| Fixed steps | Variable frame time |
|---|---|
| physics, gameplay simulation, networking ticks, anything deterministic | rendering, UI animation, camera smoothing, visual-only particles |

Systems in fixed steps read the **step**, never the frame time; systems in the frame read the **frame time**, plus
`alpha` when they display simulation state. Mixing them up (frame time inside a fixed step, or step-based motion in a
camera) is a common source of jitter.
