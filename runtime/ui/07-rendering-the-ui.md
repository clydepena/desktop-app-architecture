# Rendering the UI

How a custom-drawn toolkit's widgets become pixels, without the toolkit depending on the renderer.

## Draw data

The toolkit produces **draw data**, not GPU calls:

```
draw list:
    vertices, indices                 (rectangles, rounded corners, lines, glyph quads)
    textures                          (glyph atlases, icon atlases, images)
    commands: [ clip rectangle, texture, index range ], [ ... ], ...
```

A small **rendering backend** turns that into **one render pass**, drawn after the scene into the window's output
([Using the renderer](../rendering/06-using-the-renderer.md)). The toolkit never calls the GPU itself, so the same
toolkit runs on any renderer, or on none (in tests).

## Batching

A whole UI should take a handful of draw calls:

- **atlases** for glyphs and icons, so many widgets share one texture
- commands with the same texture and clip rectangle merged
- images drawn through the same atlas or array textures where possible

## When to redraw

| Kind of application | Redraw |
|---|---|
| **games** | every frame, along with the scene |
| **desktop applications and editors** | **only when something changed**: input, a running animation, a data change, a timer |

An editor that redraws its whole UI at 60 frames per second while nothing happens burns a CPU core and drains laptop
batteries. Immediate-mode toolkits redraw when the application asks (after input or a change); retained toolkits track
**dirty regions** and repaint only those ([The frame loop and pacing](../time/02-frame-loop-and-pacing.md)).

## Several windows

Each window renders its UI into **its own output** ([Several windows](../windowing/04-several-windows.md)). The draw data
is produced per window; textures (atlases, icons) are shared.

## Viewports inside the UI

An editor viewport panel shows a **render target's texture** as an image widget. The renderer draws the scene into the
target; the UI only displays the result and passes mouse positions (converted to pixels) back for picking
([Windowing and rendering](../windowing/06-windowing-and-rendering.md)).

## World-space UI

In-game screens, signs and 3D panels: the UI renders into a **render target** used as a texture in the scene. Input is
mapped back from the 3D surface to UI coordinates (a ray hits the panel; the hit point becomes a UI pointer position).

## Native and web toolkits

Native widgets are drawn by the OS; web views by their browser engine. Combining them with GPU content means placing
OS-drawn areas next to (or over) GPU surfaces, which is where most of their integration problems come from.
