# Several windows

Editors and tools often use more than one window: tool palettes, detached panels, a viewport on a second monitor,
a launcher before the main window, preview windows, "play in a separate window".

## Each window is independent

- its own size, position, **content scale** and **monitor**
- its own events (resized, focus, close requested)
- its own **render output** if it shows GPU content ([Windowing and rendering](06-windowing-and-rendering.md))

All windows share one GPU device and its resources (textures, meshes), so a texture loaded once can be shown in every
window.

## Multi-viewport UI

Some UI toolkits let a panel be dragged **outside** the main window, where it becomes its own OS window. The toolkit
asks the windowing service (through its platform backend) to create, move and destroy those windows, and renders each
through its own output. To the rest of the application, they are ordinary windows.

## Closing is not quitting

Closing a tool window just closes it. The lifecycle decides what closing the **main** window means (usually: quit,
after the unsaved-changes check), and what happens to the other windows then (closed together).

## Focus between windows

Keyboard focus moves between windows; input routing follows it. Shortcuts that should work "anywhere in the
application" are routed per application, not per window ([Actions](../../tool-framework/05-actions.md)).

## Saving placement

Window positions, sizes, states and monitors are **UI state** ([UI state](../../tool-framework/09-ui-state.md)), saved
per user. Restoring them needs care, because the saved setup may no longer exist:

- the saved monitor was disconnected, or its resolution or scale changed
- the window would end up partly or fully **off-screen**

So restoring **validates**: the placement is clamped to the work area of a monitor that still exists, or falls back to
a default size centered on the primary monitor. This is the **Memento** pattern with validation on restore.
