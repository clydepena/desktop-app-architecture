# The frame pipeline

Given the render world, what gets drawn, in which order, and how cheaply? Every frame goes through the same
steps, once per **view**.

## Views

A **view** is one rendering of the world: a camera (where from), a render target (where to) and settings (which
layers, which passes). A frame usually has several:

- the main camera
- shadow maps (a view from each light)
- reflections, minimaps, picture-in-picture
- in an editor: every viewport, plus thumbnails being generated

Views are cheap to add because they share the render world: proxies are stored once and seen by many views.

## 1. Culling

Skip proxies the view can't see:

- **frustum culling:** compare each proxy's bounding box with the camera's visible area
- **spatial structures** (grids, quadtrees, bounding volume hierarchies) so big worlds don't test every proxy
- **layer masks:** a view only sees the layers it asks for (an editor-only layer for gizmos, a UI layer)
- **occlusion culling** (3D): skip things hidden behind others

## 2. Draw items and sort keys

Each visible proxy becomes one or more **draw items** with a **sort key**: a single 64-bit number with the sorting
criteria packed in, most important first:

```
| layer (8 bits) | translucency (1) | depth (24) | pipeline (16) | material / texture (15) |
```

Sorting draw items by this number does two jobs at once:

- **correct order:** layers first, transparent things after opaque ones, transparent ones back to front
- **fewer state changes:** items with the same pipeline and texture end up next to each other

For opaque objects, depth is usually sorted front to back (so the GPU can skip hidden pixels); for transparent
ones, back to front (so blending is correct). The key layout can flip the depth bits per category.

## 3. Batching and instancing

Draw calls have a fixed cost, so the renderer merges compatible items:

- **batching:** items that share a texture and pipeline are merged into one draw call. In 2D, this is sprite
  batching; **texture atlases** (many images in one texture) let more sprites share one batch.
- **instancing:** many copies of one mesh drawn in one call, each with its own transform from a buffer.

## 4. Passes

A frame is drawn in **passes**, each writing into one or more targets:

```
shadow maps -> depth pre-pass -> opaque -> sky -> transparent -> post-processing -> UI -> editor overlays
```

Passes are where features plug in: adding bloom, outlines or a debug view means adding a pass.

## The render graph

Modern renderers describe the frame as a **graph of passes** instead of a hand-written sequence. Each pass
declares what it reads and writes:

```
graph.addPass("bloom", pass -> {
    pass.read(hdrColor);
    pass.write(bloomTarget);
    pass.execute(cmd -> ...);
});
```

From those declarations, the graph:

- **orders the passes** (a pass runs after the passes that write what it reads)
- **inserts GPU synchronization** (barriers) between them automatically
- **reuses memory** for targets that are never needed at the same time
- **skips passes** whose output nobody reads

It is a **directed acyclic graph** built with a **Builder**. Frostbite's "frame graph" popularized it; many engines
now use the idea. The benefit: adding or removing an effect is adding or removing a pass, without editing a long
sequence and its synchronization by hand.

## Render paths

The overall strategy for lighting is itself interchangeable (the **Strategy** pattern):

- **forward:** each object is lit as it is drawn; simple, good for few lights and for transparency
- **deferred:** first write surface properties to several targets, then light the screen once; good for many lights
- **2D:** usually forward, with sorting layers and optional 2D lights

A renderer may offer several paths behind one interface, chosen by settings or per view.
