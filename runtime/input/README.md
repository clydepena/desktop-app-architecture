# Input

How an input service is built, and how the rest of an application uses it without being tied to devices.

Input is the odd one out among the runtime services. [Rendering](../rendering/README.md) and
[audio](../audio/README.md) carry information **out of** the application; [networking](../networking/README.md)
goes both ways; input carries information **in**, from the user. It follows the same shape (a platform layer, an
abstraction over devices, a narrow interface, the same dependency direction), but its central problem is
**translation and routing**: "key 0x1E went down" must become "the player wants to move left", and reach **the
right part of the app** (a text field, a menu, the editor, or the game), exactly once.

## The layers inside an input service

```
+----------------------------------------------------------------+
| Consumers          gameplay, UI navigation, editor tools,        |  owned by the app
|                    cameras                                       |
+----------------------------------------------------------------+
| Actions            logical actions: "Jump", "Move" (2D),         |  what the app asks for
|                    "Confirm"; read per frame or tick             |
+----------------------------------------------------------------+
| Action mapping     bindings, contexts (action maps), modifiers,  |  data, rebindable
|                    interactions (hold, double tap), composites   |
+----------------------------------------------------------------+
| Routing and focus  who receives input: UI, focused panel,        |  chain of responsibility
|                    viewport, game; capture and consumption       |
+----------------------------------------------------------------+
| Device layer       per-device state and events, timestamps,      |  normalized, device-independent
|                    deadzones, connect/disconnect, players        |
+----------------------------------------------------------------+
| Platform           OS events and APIs: keyboard, mouse, gamepads,|  platform layer
|                    touch, pen, sensors, text input / IME          |
+----------------------------------------------------------------+
| OS, drivers, hardware                                              |
+----------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Platform and devices](01-platform-and-devices.md) | keyboards (scancodes vs keycodes), mice, gamepads, touch, pen, sensors |
| 2 | [The device layer and players](02-device-layer-and-players.md) | normalized state, events, deadzones, connect/disconnect, device ownership |
| 3 | [Routing and focus](03-routing-and-focus.md) | who gets an event: modals, text fields, UI, game; consumption, capture, focus loss |
| 4 | [Action mapping](04-action-mapping.md) | actions instead of keys: bindings, composites, interactions, contexts, virtual devices |
| 5 | [Timing](05-timing.md) | per-tick snapshots, latching presses, frame-rate independence, input as network data |
| 6 | [Text input](06-text-input.md) | why text is not keys: characters, dead keys, IME composition |
| 7 | [Feedback and prompts](07-feedback-and-prompts.md) | rumble, lights, cursor control, button prompts per device |
| 8 | [Accessibility](08-accessibility.md) | rebinding, hold vs toggle, timings, alternative devices |
| 9 | [Using the input service](09-using-the-input-service.md) | the public interface, and how each part of an app uses it |
| 10 | [Coupling: what may depend on input](10-coupling.md) | safe and unsafe dependencies, injected and recorded input |
| 11 | [Assets](11-assets.md) | which assets each layer uses, and what is a user setting instead |

## Rendering vs audio vs networking vs input

| | Rendering | Audio | Networking | Input |
|---|---|---|---|---|
| Information flows | out | out | out and in | **in** |
| Platform layer | RHI over GPU APIs | device layer | sockets, TLS, HTTP | OS input events and APIs |
| Driven by | the main loop | the device callback | message arrival and ticks | OS events, sampled per frame or tick |
| Main problem | drawing efficiently | never missing a deadline | an unreliable, untrusted remote side | translating and routing to the right consumer |
| Abstraction over hardware | render proxies, handles | emitters, voices | replicated objects, remote proxies | **actions** over devices; virtual devices |
| Data-driven layer | materials, render graph | sound events, mixer | replication settings, schemas | bindings, contexts, interactions |
| Null version | draws nothing | plays nothing | offline, loopback | injected or recorded input |

## Patterns in an input service

| Pattern | Where |
|---|---|
| Adapter / Bridge | the platform layer over each OS's input APIs; actions as adapters between devices and intentions |
| Chain of responsibility | routing an event from the most specific receiver outwards |
| Strategy | interactions (press, hold, tap, double tap) and processors (invert, scale, curves) |
| Composite | composite bindings (WASD forming one 2D value) |
| State (stack) | contexts / action maps pushed and popped ("Gameplay", then "Menu" on top) |
| Observer | action events, device connected / lost |
| Command | action snapshots as messages (networking, replay) |
| Memento | per-tick input snapshots, recorded and replayed |
| Null object / virtual device | injected input for tests, bots and replays |

## Related

- [Time and scheduling](../time/README.md): clocks, the frame loop, fixed steps and update order that every service runs on.
- [Prediction and timing](../networking/05-prediction-and-timing.md): input snapshots as network messages.
- [IO](../io/README.md): files, the virtual file system, packages and streaming, under every service's assets.
- [Windowing and display](../windowing/README.md): windows, displays and scale; hands surfaces to rendering through the application shell.
- [Localization](../localization/README.md): every language and region: keys, formatting, text layout and localized assets.
- [UI toolkit](../ui/README.md): widgets, layout, focus and styling on top of windowing, rendering, input and localization.
- [Settings](../04-settings.md): where user bindings and preferences are stored.
- [Actions](../../tool-framework/05-actions.md): editor shortcuts and keybindings, the tool-layer counterpart of
  action mapping.
- [Selection and context](../../tool-framework/06-selection-and-context.md): focus and context keys used by routing.
