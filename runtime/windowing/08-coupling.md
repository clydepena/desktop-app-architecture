# Coupling: what may depend on windowing

The direction rule: consumers depend on the windowing service's interface; the windowing service depends only on the
platform layer. Upward communication is through events, delivered on the main thread.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle / application shell** | creates windows, mediates between windowing and rendering, routes close requests |
| **UI toolkit** | lays out inside windows; its platform backend creates extra windows |
| **Input** | receives window events; acts on windows for cursor control |
| **Time** | reads window state and refresh rates for pacing |
| **UI state, settings** | placement, display mode, monitor choice |
| **Rendering, through surfaces only** | consumes opaque native surfaces handed over by the shell; never the window itself |

## Keep decoupled

| System | How it gets what it needs instead |
|---|---|
| **Rendering and windowing, both ways** | never call each other; the shell passes surfaces, sizes and events ([Windowing and rendering](06-windowing-and-rendering.md)) |
| **Domain / gameplay** | never touches windows; reads view sizes and aspect ratios from its camera and view data |
| **Documents, undo** | close requests go through the lifecycle's check; windows don't save documents |
| **Audio, networking, IO** | unrelated to windows; IO receives dropped paths as plain data |

## Mistakes to avoid

- **The renderer creating or owning windows**, or window code calling GPU APIs (outside the OpenGL backend's hidden
  necessity).
- **Logical size used as pixel size:** blurry rendering, tiny UI, wrong picking on high-DPI displays.
- **Reading the content scale only at startup:** it changes when a window moves to another monitor.
- **Recreating a swapchain at 0 x 0** when minimized, or recreating it on every resize event.
- **Destroying a window before its swapchain.**
- **Assuming one window and one monitor.**
- **Exclusive fullscreen that doesn't restore the display mode** on exit or crash.
- **Restoring saved placement off-screen** after a monitor was removed.
- **Touching windows from worker threads.**
- **Blocking inside the OS's modal loops** (resizing, blocking dialogs) without a plan for frames during them.

## Testing the boundary

- **Headless rendering tests:** render offscreen and compare with golden images, with no window at all.
- **A null or hidden window** for tests that need window events without a visible window.
- **Fake display configurations:** two monitors at different scales, a monitor removed, an HDR and a standard monitor,
  to test placement restore and scale changes without the hardware.
