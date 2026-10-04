# Windows, state and events

## What a window has

- a **title**, an **icon**, **decorations** (borders and a title bar, or none)
- a **position** and a **size**, with optional minimum and maximum sizes, resizable or not
- a **state**: normal, minimized, maximized, fullscreen; visible or hidden
- **focus**: whether it receives keyboard input
- a **content area**, the part the application draws into (the window minus its borders and title bar)

## Events

Changes arrive as events (the **Observer** pattern). Each has consumers that care:

| Event | Who reacts |
|---|---|
| **close requested** | the lifecycle: asks about unsaved changes, and may cancel. It is a *request*, not a fact ([Lifecycle](../01-lifecycle.md)) |
| **resized** (logical size and pixel size) | the UI re-lays out; the renderer resizes that window's output |
| **content scale changed** (the window moved to another monitor) | the UI rebuilds fonts and layout; the renderer may resize the output |
| **minimized / restored, occluded** | rendering stops or slows for that window; pacing drops to background rates |
| **focus gained / lost** | input releases held keys ([Routing and focus](../input/03-routing-and-focus.md)); pacing may change |
| **moved / display changed** | placement is saved; refresh rate and HDR capabilities are read again |
| **files dropped** | dropped paths go to the import or open logic ([IO](../io/README.md), [Documents](../../tool-framework/02-documents.md)) |
| **redraw requested** | in event-driven apps, the OS asks for the content to be drawn again |

## Rules

- **Close is a request.** The close button, Alt+F4, the OS logging off and "Quit" all ask; the application decides,
  after the unsaved-changes check.
- **React to state, don't assume it.** A window can be minimized, moved to another monitor or resized at any moment,
  including while a modal dialog is open.
- **Events can arrive in bursts** (a drag produces many resize events). Consumers apply the **latest** value at a
  frame boundary rather than doing heavy work per event.
