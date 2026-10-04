# Displays, scale and modes

## Monitors

Each monitor (display) has:

- a **position** in the desktop (monitors are arranged side by side, possibly with different sizes)
- a **work area**: the monitor minus taskbars and docks, where windows should be placed
- a **resolution** and a **refresh rate** (60, 120, 144 Hz...), possibly variable
- a **content scale** (100%, 125%, 150%, 200%...) chosen by the user
- **color capabilities**: standard or HDR, color space, brightness range

Monitors are added, removed and reconfigured while the application runs; a "displays changed" event says so.

## Logical units and physical pixels

The most common confusion in this area: a window has two sizes.

| Unit | Meaning | Used by |
|---|---|---|
| **logical units** (points, "device-independent pixels") | size as the user perceives it: the same physical size on every monitor | window sizes in settings, UI layout, fonts, mouse positions from many platforms |
| **physical pixels** (the framebuffer size) | the actual pixels on screen | the renderer, swapchains, render targets, picking against id buffers |

```
pixels = logical units x content scale
```

A 1280 x 720 window at 200% scale is **2560 x 1440 pixels**.

- Rendering at the logical size gives a blurry, upscaled image.
- Laying out UI in pixels gives tiny text on high-DPI monitors.
- Picking with logical mouse coordinates against a pixel-sized id buffer selects the wrong object.

Every value crossing between the UI, input and rendering says which unit it is in, and is converted with the
window's current content scale.

## Per-monitor scale

Dragging a window from a 100% monitor to a 200% monitor changes its scale in the middle of a session. The application
receives a **content scale changed** event and must re-layout, rebuild fonts at the new size and resize its render
output. Reading the scale only at startup is a classic bug: text looks blurry or tiny after moving a window.

## Display modes

| Mode | What it is | Trade-offs |
|---|---|---|
| **Windowed** | a normal window | everything else on the desktop keeps working |
| **Borderless fullscreen** | a window without decorations covering one monitor, at the desktop's resolution | instant switching between applications, overlays and notifications work; the common default today |
| **Exclusive fullscreen** | the application takes over the display and may change its resolution | historically lower latency; slow switching; the display mode must be restored on exit, **even after a crash** |

## Resolution vs render resolution

In borderless fullscreen, the window is the monitor's resolution. The renderer may still render at a lower **render
scale** (e.g. 75%) and upscale the result, to gain speed. That is a **rendering** decision, not a display mode: the
window, the swapchain and the UI stay at full resolution, and only the 3D or 2D scene is rendered smaller.

## HDR and color

A monitor's HDR capability and color space are reported by the windowing service; the renderer chooses its output
format and tone mapping from them ([Windowing and rendering](06-windowing-and-rendering.md)). A window moved between
an HDR and a standard monitor changes what is possible, so this is re-checked on "moved" and "displays changed".
