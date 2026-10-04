# Windowing and rendering

The two services meet at the screen, and they are easy to tangle. This page untangles them: what belongs to which,
how they are connected without depending on each other, and how each works with or without the other.

## Four terms that get confused

| Term | What it is | Owned by |
|---|---|---|
| **Window** | the OS object: title, borders, position, focus, events | windowing |
| **Surface** | an opaque handle to "this window's content area, for the GPU to present into" | created by windowing, **consumed** by the rendering backend |
| **Swapchain** (an *output*) | a few GPU images drawn into and shown on the surface in turn; its size is the **pixel** size; its *present mode* decides vsync | rendering |
| **Render target / view** | offscreen images and cameras. An editor's "viewport panel" is a UI panel showing a render target's texture, **not** a window | rendering |

## Who owns which concern

| Concern | Owner |
|---|---|
| creating, moving, sizing windows; fullscreen modes | windowing |
| the pixel size and content scale | windowing **reports** them |
| recreating the swapchain at a new size | rendering, when told |
| **vsync** | a **policy** from settings and the time service's pacing; **implemented** as the swapchain's present mode by rendering; **informed** by the display's refresh rate from windowing |
| HDR output | windowing reports the display's capabilities; rendering picks the output format and tone mapping |
| render resolution / render scale | rendering |

## Siblings, joined by a mediator

**Neither service depends on the other.** Both sit just above the platform layer, side by side:

```
            application shell / lifecycle      <- knows both, connects them (Mediator)
              /                      \
   windowing and display          rendering
   (windows, displays, events)    (RHI, outputs, views)
              \                      /
         platform: the native surface handle type (opaque, shared vocabulary)
```

- **Windowing knows nothing about GPUs.** It produces a native surface handle: an opaque value, a stand-in like any
  other handle.
- **Rendering knows nothing about windows.** It accepts a native surface and makes an output from it
  ([The RHI](../rendering/01-rhi.md)). It never asks for titles, positions or focus.
- **The application shell (or the lifecycle) connects them:** it creates a window, passes its surface to the renderer,
  forwards resize and scale events to the right output, and closes them in the right order. This is the **Mediator**
  pattern: two components that don't know each other, coordinated by a third that knows both.

```
window = windowing.create(mainWindowDesc)
output = renderer.createOutput(windowing.nativeSurface(window))

windowing.onResized(window, size -> renderer.resizeOutput(output, size.pixels))
windowing.onMinimized(window, ()  -> renderer.pauseOutput(output))
windowing.onClose(window, ()      -> { renderer.destroyOutput(output); windowing.destroy(window); })
```

## The OpenGL exception

In OpenGL, a rendering context is tied to a window (or to its pixel format), so many libraries create the **window and
the context together** in one call. That makes windowing and rendering *look* inseparable. Vulkan, Direct3D 12 and
Metal separate them cleanly: a device first, surfaces whenever a window needs one.

The fix is to **hide the quirk inside the OpenGL backend of the rendering hardware interface**: the backend creates its
context however OpenGL requires (with a hidden window, or a surfaceless context where supported) and accepts real
windows' surfaces later. The rest of the architecture keeps the clean split.

## With or without each other

| Combination | Examples | How it works |
|---|---|---|
| **Both** | a normal editor or game | the shell creates windows; the renderer creates one output per window that shows GPU content |
| **Rendering without windowing** (headless / offscreen) | thumbnails, servers producing images or video, tests comparing rendered images with reference ("golden") images, command-line exporters | the renderer creates a device and offscreen targets, renders, and reads the pixels back or encodes them. No surface, no swapchain, no presenting. With OpenGL, a surfaceless or hidden-window context |
| **Windowing without rendering** | apps drawn with the OS's own widgets or a CPU 2D library; a server with a status window; tests of window behavior | windows exist and deliver events; drawing goes through the OS or a software renderer that copies pixels to the window, or a null renderer leaves it cleared |
| **Neither** | command-line tools, dedicated servers, most automated tests | no windows, a null renderer; the domain runs alone |

That every row works is the evidence that the split is right. If the renderer couldn't start without a window,
thumbnails, servers and image-comparison tests would all need fake windows.

## Edge cases between them

- **Minimized window:** the pixel size becomes 0 x 0. Never create a swapchain of size zero (a common crash): **skip
  rendering and presenting** for that output until it is restored.
- **Resize storms:** a drag produces many resize events per second. Recreate the output **at a frame boundary**, with
  the latest size, not once per event.
- **Surface lost** (the window was destroyed, the display reconfigured): the renderer recreates the output when the
  shell gives it a new surface.
- **Device lost** (GPU reset, driver update): the renderer recreates its resources and outputs; the windows are
  untouched.
- **Closing order:** destroy an output (its swapchain) **before** its window; destroy outputs and GPU resources before
  the device.
- **Threads:** windows live on the main thread. If presenting happens on a render thread, size and scale changes are
  **handed over** as messages and applied at the next frame boundary; the window object itself is never shared
  between threads.

## The neighbors

- **Time:** pacing uses the display's refresh rate and each window's state (background, minimized); vsync is the
  pacing policy, carried out by the swapchain ([The frame loop and pacing](../time/02-frame-loop-and-pacing.md)).
- **Input:** mouse coordinates usually arrive in **logical** units and are converted with the content scale before
  picking against pixel buffers ([Displays, scale and modes](03-displays-scale-and-modes.md)).
- **UI toolkit:** lays out in logical units, draws through the renderer into the window's output, and in
  multi-viewport mode asks windowing for extra windows ([Several windows](04-several-windows.md)).
