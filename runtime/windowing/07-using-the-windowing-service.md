# Using the windowing service

How the rest of an application uses windows and displays. Callers create and control windows through a few doorways
and react to events; the windowing service never calls into rendering, the UI or the domain.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Windows** | create and control | `create(desc) -> WindowId`, `destroy(id)`, `show` / `hide`, `setTitle`, `setSize(logical)`, `setPosition`, `minimize` / `maximize` / `restore`, `focus` |
| **Properties** | read size and scale | `size(id)` (logical), `pixelSize(id)`, `contentScale(id)`, `monitorOf(id)`, `state(id)` |
| **Display modes** | fullscreen | `setMode(id, WINDOWED / BORDERLESS / EXCLUSIVE, monitor, resolution)` |
| **Displays** | monitors | `monitors()`, `workArea(m)`, `scale(m)`, `refreshRate(m)`, `hdrCapabilities(m)`, `modes(m)` |
| **Surfaces** | for rendering | `nativeSurface(id) -> NativeSurface` (opaque) |
| **Events** | what happened | `closeRequested`, `resized`, `scaleChanged`, `focusChanged`, `minimized`, `restored`, `moved`, `displaysChanged`, `filesDropped` |
| **Placement** | save and restore | `placement(id) -> Placement`, `restore(id, placement)` (validated against current monitors) |
| **OS integration** | the rest | `setIcon`, `setTaskbarProgress`, `requestAttention`, `openFileDialog(...) -> Future<Path>`, `clipboard()`, `systemTheme()`, `openUrl` |

## How each part of an application uses it

| Part | How it uses windowing |
|---|---|
| **Lifecycle / application shell** | creates the main window early ("show the window early"), connects its surface to the renderer, routes **close requested** through the unsaved-changes check, forwards resize and scale events to outputs, closes windows (outputs first) at shutdown ([Lifecycle](../01-lifecycle.md)) |
| **UI toolkit** | lays out in logical units, rebuilds fonts on scale changes, creates extra windows for detached panels |
| **Renderer** | owns one output per window that shows GPU content; sees only surfaces and pixel sizes ([Windowing and rendering](06-windowing-and-rendering.md)) |
| **Input** | receives events through windows, releases keys on focus loss, confines or locks the cursor on a window ([Input](../input/README.md)) |
| **Time** | throttles pacing for minimized or unfocused windows; reads refresh rates ([Time and scheduling](../time/README.md)) |
| **UI state** | saves and restores window placement and monitors ([UI state](../../tool-framework/09-ui-state.md)) |
| **Settings** | display mode, monitor, resolution (read by windowing); vsync and render scale (read by time and rendering) ([Settings](../04-settings.md)) |
| **Documents and IO** | receive dropped files and file-dialog results |

## Editors

- **Several windows and detached panels**, each with its own scale and output ([Several windows](04-several-windows.md)).
- **A launcher** that becomes the main editor window (resized, restyled), or a separate small window that closes when
  the editor opens.
- **Per-window layouts**, saved as UI state.
- **"Play in a separate window"** to test at a real resolution and aspect ratio, independent of the editor's layout.
