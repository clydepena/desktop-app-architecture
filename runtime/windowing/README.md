# Windowing and display

How an application gets a place on the screen: OS windows with titles, borders, positions, sizes and focus, on
monitors with their own resolutions, scaling and refresh rates. It is the service rendering **presents into**, input
**receives events from**, and the UI **lays out inside**.

It has the familiar shape (a platform layer, an abstraction over OS windows and displays, events, a narrow
interface, the same dependency direction). Its special problems are **units and change**: sizes in logical units vs
physical pixels, monitors with different scaling, windows resized, minimized, moved between monitors or closed at
any moment, and operating systems with strict rules about which thread may touch windows.

A large part of this series is about its relationship with [rendering](../rendering/README.md): the two are
**siblings that never call each other**, joined by the application shell. See
[Windowing and rendering](06-windowing-and-rendering.md).

## The layers inside a windowing service

```
+-------------------------------------------------------------------+
| Consumers          application shell, UI toolkit, renderer outputs, |  owned by the app
|                    input, tools                                     |
+-------------------------------------------------------------------+
| Window manager     window objects, several windows, state           |  the service
|                    (size, position, minimized, fullscreen), events,  |
|                    placement saved and restored                     |
+-------------------------------------------------------------------+
| Displays           monitors, work areas, scale (DPI), refresh rate, |
|                    color and HDR capabilities, display modes        |
+-------------------------------------------------------------------+
| OS integration     title, icon, taskbar, notifications, file        |
|                    dialogs, drag and drop, clipboard, system theme   |
+-------------------------------------------------------------------+
| Platform           OS window APIs (Win32, Cocoa, X11 / Wayland),     |  platform layer
|                    the message pump, native surface handles          |
+-------------------------------------------------------------------+
| OS window manager / compositor, display hardware                       |
+-------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform and the message pump](01-platform-and-message-pump.md) | OS window APIs, pumping messages, thread rules, modal loops |
| 2 | [Windows, state and events](02-windows-state-and-events.md) | what a window is, its states, and who reacts to each event |
| 3 | [Displays, scale and modes](03-displays-scale-and-modes.md) | monitors, logical units vs pixels, per-monitor scale, fullscreen modes |
| 4 | [Several windows](04-several-windows.md) | tool windows, detached panels, multi-viewport UI, placement |
| 5 | [OS integration](05-os-integration.md) | icons, taskbar, dialogs, drag and drop, clipboard, system theme |
| 6 | [Windowing and rendering](06-windowing-and-rendering.md) | window vs surface vs swapchain; siblings joined by a mediator; with or without each other |
| 7 | [Using the windowing service](07-using-the-windowing-service.md) | the public interface, and how each part of an app uses it |
| 8 | [Coupling: what may depend on windowing](08-coupling.md) | safe and unsafe dependencies, headless tests, fake displays |
| 9 | [Assets](09-assets.md) | window definitions, icons, cursors, splash screens, display presets |

## How windowing compares

| | Rendering | Input | Time | Windowing and display |
|---|---|---|---|---|
| Role | draws images | carries information in | the heartbeat | **a place on screen to show them** |
| Platform layer | GPU APIs | OS input | OS clocks | **OS window and display APIs** |
| Driven by | the main loop | OS events | itself | **OS messages, pumped by the main loop** |
| Abstraction | proxies, outputs | actions | clocks | **windows, displays, opaque surfaces** |
| Relationship to rendering | n/a | none | pacing, vsync policy | **sibling, joined by the shell; hands over surfaces** |
| Null version | draws nothing | injected input | manual clock | **no window or a hidden window; headless rendering** |

## Patterns in a windowing service

| Pattern | Where |
|---|---|
| Adapter / Bridge | one interface over each OS's window APIs |
| Abstract Factory | a platform backend that creates windows, cursors and dialogs for that OS |
| Observer | window and display events |
| Mediator | the application shell joining windowing and rendering, which don't know each other |
| Opaque handle | the native surface handed from windowing to rendering |
| Memento (with validation) | saved window placement, clamped to monitors that still exist |
| Null object | no window or a hidden window; headless rendering |

## Related

- [Time and scheduling](../time/README.md): pacing, vsync policy, background throttling.
- [Rendering](../rendering/README.md): presents into windows through surfaces and swapchains.
- [Input](../input/README.md): receives raw input and focus through windows.
- [UI toolkit](../ui/README.md): widgets, layout, focus and styling on top of windowing, rendering, input and localization.
- [Lifecycle](../01-lifecycle.md): creates the main window and handles close requests.
- [UI state](../../tool-framework/09-ui-state.md): where window placement is saved.
