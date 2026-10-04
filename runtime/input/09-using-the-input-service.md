# Using the input service

How the rest of an application receives input. Consumers ask for **actions** (or subscribe to action events); they
never talk to devices. The input service never calls into gameplay, the UI or the editor: consumers pull from it.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Actions** | read intentions | `value("Move") -> Vector2`, `pressedThisTick("Jump")`, `releasedThisTick("Fire")`, `isHeld("Aim")` |
| **Action events** | react to actions | `on("Jump", started / performed / canceled, handler)` |
| **Contexts** | switch what is active | `push("Menu")`, `pop()`, `enable("Vehicle")` |
| **Bindings** | rebinding screens | `listenForNextInput(action) -> Future<Binding>`, `setBinding(action, binding)`, `resetToDefaults()` |
| **Devices and players** | who is using what | `devices()`, `assign(device, player)`, events `deviceConnected`, `deviceLost` |
| **Text input** | typing | `beginTextInput(caretRect)`, character and composition events, `endTextInput()` |
| **Cursor** | pointer control | `lockCursor()`, `confine()`, `setCursorShape(shape)` |
| **Feedback** | output on input devices | `rumble(player, pattern)`, `setLight(device, color)` |
| **Prompts** | what to show the user | `glyphFor(action, player) -> Icon` |
| **Injection** | simulated input | `inject(device, event)`, virtual devices for tests, replays, bots and remote input |

## How each part of an application uses it

### The lifecycle

The main loop ([Lifecycle](../01-lifecycle.md)) pumps OS events at the start of each frame, builds the input
snapshot before the simulation runs ([Timing](05-timing.md)), and releases all held input when the window loses
focus.

### The UI toolkit

Receives raw input **first**, through routing ([Routing and focus](03-routing-and-focus.md)). It handles typing,
clicks, scrolling and keyboard or gamepad navigation, and reports what it consumed.

### Gameplay and the domain

Read **actions** only, per tick, from the snapshot. A player controller turns "Move" into movement, "Jump" into a
jump. Gameplay never names keys, buttons or devices.

### Cameras and tools

Cameras read "Look" and "Zoom". In an editor, the editor camera has its own context ("Editor camera"), so its
controls never clash with a game running in the same window.

### Settings

Bindings, sensitivity, inversion, deadzones and hold/toggle choices are **user settings**
([Settings](../04-settings.md)), layered over the shipped defaults and applied live.

### Networking

Sends **action snapshots** per tick, never raw key events
([Prediction and timing](../networking/05-prediction-and-timing.md)). The server validates the intentions they
carry, like any other client message.

### Diagnostics

An **input debugger** shows device states, the active context stack, which binding triggered which action, and which
receiver consumed an event ([Diagnostics](../05-diagnostics.md)). Most "the button doesn't work" bugs are found
there in seconds.

### Editors

- **Editor shortcuts** are tool-framework actions ([Actions](../../tool-framework/05-actions.md)): the same idea as
  action mapping, living in the tool layer and routed **before** game input.
- **Playing inside the editor:** the running game receives input only while its viewport is focused. A dedicated
  key releases a locked cursor back to the editor, and editor shortcuts stay reachable.
- **Input recording and playback:** record a session's action snapshots and replay them to reproduce bugs.
- **A binding editor** for the default bindings asset, with conflict detection.
