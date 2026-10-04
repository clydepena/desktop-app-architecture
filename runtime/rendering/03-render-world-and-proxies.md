# The render world and render proxies

The most important idea in renderer architecture: **the renderer doesn't draw the application's objects
directly**. Each drawable thing in the application has a **render proxy**, a small renderer-owned record holding
only what is needed to draw it. All proxies together form the **render world**.

| Application object | Its render proxy |
|---|---|
| a game object with a sprite | transform matrix, texture handle, texture rectangle, color, sort layer |
| a 3D model | mesh handle, material handle, world matrix, bounding box |
| a light | type, color, intensity, position, shadow settings |
| a camera or viewport | view and projection matrices, render target, which layers it sees |
| a text label | glyph quads, font atlas handle, color |

Real engines all have this split, under different names: Unreal Engine's scene proxies, Bevy's separate render
world filled by an "extract" step, Godot's rendering server that only knows objects by id.

## Why proxies

- **Decoupling.** The renderer knows nothing about game objects, documents, components or UI widgets. The same
  renderer draws a game, an editor viewport, a thumbnail and a UI, because all of them only produce proxies.
- **Threading.** The application can update its objects for the next frame while the renderer draws the previous
  one from its proxies. Neither touches the other's data, so no locks are needed.
- **Speed.** Proxies are compact and stored in arrays, so culling and sorting walk through memory in order
  (data-oriented design) instead of chasing pointers through application objects.
- **A consistent frame.** Everything drawn in one frame comes from one snapshot, never from objects half-way
  through being updated.

## Keeping proxies in sync

Two common approaches.

### Extract every frame

Each frame, copy the needed data from every visible object into the render world, then throw it away after
drawing.

- simple: no bookkeeping, no "forgot to update the proxy" bugs
- costs a copy per object per frame, fine for small and medium scenes

### Create once, then send changes

```
object appears   -> create its proxy            renderer.createSprite(desc) -> id
object changes   -> send only what changed      renderer.setTransform(id, matrix)
object removed   -> remove its proxy            renderer.destroy(id)
```

- cheaper for big scenes where most things don't move
- needs a **dirty flag** per object, so only changed objects send updates
- the proxy id is stored next to the object, by whatever connects the two (see the bridge in
  [6](06-using-the-renderer.md))

Many renderers mix both: static things are created once, fast-changing things (particles, UI) are extracted every
frame.

## What does not belong in a proxy

- Game logic, behavior, references to other application objects.
- Anything the renderer would have to call back into the application to compute.

If drawing an object requires asking the object something every frame, that question's answer belongs in the
proxy instead.
