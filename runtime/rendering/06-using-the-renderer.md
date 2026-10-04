# Using the renderer

How the rest of an application gets anything on screen. The application never "renders": it **describes** what
should be visible through a few doorways, and the renderer does the rest. The renderer never calls back up into
the application; anything it needs to report goes out as events.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Render world** | create, update and remove render proxies | `createSprite(desc) -> id`, `setTransform(id, m)`, `destroy(id)` |
| **Views** | say where to render from, and into what | `createView(camera, target)`, `setLayers(view, mask)` |
| **Render targets** | get offscreen images to render into | `createTarget(w, h, format) -> TextureHandle` |
| **Resources** | hand decoded data to the GPU | `createTexture(pixels) -> handle`, `createMesh(vertices)` |
| **Immediate drawing** | shapes for one frame only | `debug.line(a, b, color)`, `debug.box(bounds)` |
| **Readback** | get pixels back, asynchronously | `readPixel(target, x, y) -> Future<int>` |
| **Features / passes** | add drawing steps (an extension point) | `registerPass(outlinePass)` |
| **Frame control** | drive the frame | `beginFrame()`, `render(views)`, `present()` |

## How each part of an application uses it

### The lifecycle drives frames

The main loop ([Lifecycle](../01-lifecycle.md)) owns the renderer and is the **only** caller of frame control:

```
startup:     renderer.init(rhi)                                   (works without any window: offscreen rendering)
window:      windowing creates a window -> native surface -> renderer.createOutput(surface)   (a swapchain)
each frame:  update the app -> sync proxies -> renderer.render(views) -> present each output
resize:      windowing reports a new size or scale -> lifecycle -> renderer.resizeOutput(output, w, h)
shutdown:    destroy outputs before their windows, free render resources before the GPU device is destroyed
```

The renderer never receives a window, only a **surface** to present to. The window and the renderer meet in the
lifecycle (or an application shell), which owns both and connects them; see [Windowing and rendering](../windowing/06-windowing-and-rendering.md).

### Resources hand over decoded data

A texture loader ([Resources](../07-resources.md)) decodes the file on a worker thread, then, in its finishing
step, calls `renderer.createTexture(pixels)`. The asset handle wraps the GPU handle. The renderer never opens
files: it only receives data that is ready to upload.

### The domain goes through bridges

Domain objects (game objects, shapes in a drawing, nodes in a scene) don't know the renderer. A small **bridge**
connects the two: a render component on the object, or a system that watches domain changes and turns them into
proxy calls.

```
object appears   -> bridge: id = renderer.createSprite(...)   (the bridge keeps the id)
object moved     -> bridge: renderer.setTransform(id, ...)    (only if dirty)
object removed   -> bridge: renderer.destroy(id)
```

The bridge is the **only** code that knows both sides. Replacing the renderer means rewriting bridges, not domain
logic; the domain can run with no renderer at all.

### The UI toolkit draws as one pass

UI libraries usually produce lists of triangles with textures ("draw data"). A small UI backend turns that into one
render pass, drawn after the scene. The UI never touches proxies.

### Editors build on views, targets, readback and passes

- **A viewport panel** is a view (with the editor's camera) plus a render target. The panel shows the target's
  texture as an image. Several viewports are several views of the same render world.
- **Selection outlines:** the selection system doesn't ask the renderer to "draw this selected". The editor
  registers an **outline pass** and gives it the ids of the selected proxies each frame. The renderer has no idea
  what "selection" means.
- **Picking:** the editor's views enable an **id pass** that writes each proxy's id into a target. On a click, the
  editor calls `readPixel(...)`, gets the id a frame later, maps it back to an object through the bridge, and
  changes the selection.
- **Gizmos, grid, bounding boxes:** immediate drawing or an overlay pass on editor views only. They never enter
  the render world, are never saved, and never appear in the shipped product.
- **Thumbnails and previews:** a job asks for an offscreen view of one asset at low priority and stores the image.
- **Hot reload and debug views:** shaders and textures are replaced behind their handles; wireframe, overdraw or
  depth views are alternative passes.

### Settings and diagnostics

- The renderer reads **its own** settings (vsync, quality, resolution scale) and listens for changes
  ([Settings](../04-settings.md)); nobody pushes settings into it.
- It publishes **statistics** (frame time, draw calls, GPU memory) for a debug panel to read, and logs through the
  normal logger ([Diagnostics](../05-diagnostics.md)).

## The same shape elsewhere

Audio and physics services are usually built the same way: the domain talks to them through bridges, they keep
their own proxies (sound sources and listeners; physics bodies), views become listeners or simulation regions, and
they report back through events. Learning this shape once covers all three.
