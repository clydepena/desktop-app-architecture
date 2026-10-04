# Rendering

How a rendering service is built, and how the rest of an application uses it without being tied to it.

The central idea: the application **never "renders"**. It **describes** what should be visible (these objects,
seen from this camera, into this image) through a small interface, and the renderer turns those descriptions into
frames. Everything else follows from keeping that boundary clean.

## The layers inside a renderer

```
+-------------------------------------------------------------+
| Scene / domain     the app's own objects: game objects,      |  owned by the app
|                    documents, UI widgets                     |
+-------------------------------------------------------------+
|        extract / sync (copy what is needed to draw)          |
+-------------------------------------------------------------+
| Render world       render proxies: sprites, meshes, lights,  |  owned by the renderer
|                    cameras, as plain render data             |
+-------------------------------------------------------------+
| Frame pipeline     views -> culling -> sorting and batching  |  what to draw, in what order
|                    -> render passes (render graph)           |
+-------------------------------------------------------------+
| GPU resources      buffers, textures, shaders, pipeline      |  handles, pools, uploads,
|                    states, render targets                    |  deferred deletion
+-------------------------------------------------------------+
| RHI                one interface over OpenGL / Vulkan /      |  platform layer
|                    DirectX / Metal                           |
+-------------------------------------------------------------+
| GPU driver                                                    |
+-------------------------------------------------------------+
```

Each layer turns something higher-level ("this sprite is at this position") into something lower-level (GPU
commands), and knows nothing about the layers above it.

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [The RHI](01-rhi.md) | one interface over the GPU APIs; command lists |
| 2 | [GPU resources](02-gpu-resources.md) | handles, pools, generations, uploads, deferred deletion |
| 3 | [The render world and render proxies](03-render-world-and-proxies.md) | the renderer's own copy of what to draw, and how it stays in sync |
| 4 | [The frame pipeline](04-frame-pipeline.md) | views, culling, sort keys, batching, passes, the render graph |
| 5 | [Frames and threads](05-frames-and-threads.md) | frames in flight, buffering, render threads, parallel recording |
| 6 | [Using the renderer](06-using-the-renderer.md) | the renderer's public interface, and how each part of an app uses it |
| 7 | [Coupling: what may depend on the renderer](07-coupling.md) | safe and unsafe dependencies, and how to test the boundary |
| 8 | [Assets](08-assets.md) | which assets each layer uses, source vs runtime formats |

Documents 1-5 describe the inside of a renderer; 6-7 describe its outside, which matters more to everyone who
isn't writing the renderer; 8 describes the data that flows in.

## Do renderers use proxies?

Yes, in two different senses, both covered here:

- **Render proxies** ([3](03-render-world-and-proxies.md)): the renderer's own small record of each object to draw,
  separate from the application's objects. This is what decouples the renderer from everything else.
- **Resource handles** ([2](02-gpu-resources.md)): stand-ins for GPU resources (the classic Proxy pattern), so the
  real resource can be created later, recreated or freed safely.

## Patterns in a renderer

| Pattern | Where |
|---|---|
| Bridge / Adapter | the RHI: one interface, one backend per GPU API |
| Command | command lists/buffers recorded now, executed by the GPU later |
| Proxy | resource handles; render proxies for scene objects |
| Flyweight | one mesh, material or texture shared by thousands of proxies |
| Object pool | GPU resources, per-frame allocations, draw lists reset every frame |
| Dirty flag | proxies update only when their object changed |
| Builder | pipeline states, render passes, the render graph |
| Directed acyclic graph | the render graph: passes ordered by what they read and write |
| Strategy | interchangeable render paths (forward / deferred), post-processing effects |
| Producer / consumer | the main thread produces frame data, the render thread consumes it |
| Double / triple buffering | per-frame data, so the CPU never overwrites what the GPU is reading |
| Immediate vs retained mode | the scene is retained (proxies persist); debug lines are immediate (one frame only) |
| Null object | a renderer that draws nothing, for servers, tools and tests |

## Related

- [Time and scheduling](../time/README.md): clocks, the frame loop, fixed steps and update order that every service runs on.
- [Memory](../memory/README.md): lifetimes, allocators, pools, budgets and GPU memory under every service.
- [Audio](../audio/README.md): the same architecture for sound.
- [Networking](../networking/README.md): the same architecture for sharing state across machines.
- [Input](../input/README.md): the runtime service that carries information in, from devices to intentions.
- [IO](../io/README.md): files, the virtual file system, packages and streaming, under every service's assets.
- [Windowing and display](../windowing/README.md): windows, displays and scale; hands surfaces to rendering through the application shell.
- [UI toolkit](../ui/README.md): widgets, layout, focus and styling on top of windowing, rendering, input and localization.
- [Lifecycle](../01-lifecycle.md): the main loop that drives each frame.
- [Jobs](../06-jobs.md): decoding and uploading on worker threads, parallel command recording.
- [Resources](../07-resources.md): the asset system; its handles wrap GPU handles for textures and meshes.
- [Inspectors](../../tool-framework/07-inspectors.md) and the rest of the tool framework: the editor features built
  on top of the renderer (viewports, picking, overlays) are described in [6](06-using-the-renderer.md).
