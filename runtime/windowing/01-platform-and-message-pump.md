# Platform and the message pump

The lowest layer: each operating system's window API (Win32 on Windows, Cocoa/AppKit on macOS, X11 or Wayland on
Linux), behind one interface (the **Adapter / Bridge** pattern). A platform **backend** creates the windows, cursors
and dialogs for its OS (the **Abstract Factory** pattern).

## The message pump

The OS delivers everything about a window (resize, move, close, focus, input, "please redraw") as **messages** in a
queue. The application must **pump** that queue regularly: take each message and handle it.

```
each frame:
    while (platform.pollMessage(out msg)) platform.dispatch(msg);   // turns OS messages into window and input events
```

An application that stops pumping for a few seconds is shown as **"not responding"** by the OS, and its windows
can't be moved or closed. This is one more reason the main thread must never block ([Time and scheduling](../time/README.md)).

When idle, the loop can **wait** for the next message instead of polling, so the application uses no CPU until
something happens ([Lifecycle](../01-lifecycle.md)).

## Thread rules

Most operating systems expect windows to be created, changed and pumped on **one thread**, usually the main thread;
macOS requires it. Consequences:

- the windowing service lives on the main thread
- other threads (a render thread, workers) never touch window objects; they send requests to the main thread
- events produced by windows are handed to other threads as messages, at frame boundaries

## Modal loops

Some OS operations run a loop **inside the OS** until they finish, and the application's own main loop doesn't run
meanwhile:

- on Windows, **dragging or resizing a window** with the mouse
- native **message boxes** and **file dialogs** (in their blocking forms)
- menus opened from the window's title bar or system menu

During these, frames stop unless the application draws from inside the OS's callbacks (for example, on each resize
message). This is the "window being dragged" stall mentioned in [the frame loop](../time/02-frame-loop-and-pacing.md):
either keep a minimal frame running from those callbacks, or accept a frozen picture and clamp the delta time
afterwards.

## Native handles

The backend can give out the OS's own handle for a window (an `HWND`, an `NSWindow`, a Wayland surface) as an
**opaque value**. Only code that needs to talk to the OS directly receives it: above all, the rendering backend, which
turns it into something it can present into ([Windowing and rendering](06-windowing-and-rendering.md)).
