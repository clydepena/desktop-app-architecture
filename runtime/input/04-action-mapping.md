# Action mapping

The key idea of an input service: **the application asks for actions, not keys.**

```
action "Jump"   (button)  <- Space, gamepad south button, touch button "jump"
action "Move"   (2D axis) <- W/A/S/D (composite), left stick, arrow keys, touch joystick
action "Look"   (2D axis) <- mouse delta (scaled), right stick (with a curve)
action "Fire"   (button)  <- left mouse button, right trigger (pressed past 0.5)
```

Gameplay asks "how much does the player want to move?" and never mentions a key, a stick or a device. New devices,
rebinding, accessibility remaps and touch controls then work without changing gameplay code.

## Actions

Each action has a **type**:

- **button:** pressed or not (with "pressed this tick" and "released this tick")
- **axis:** one value, e.g. -1 to 1 (throttle, zoom)
- **2D axis / vector:** two values (movement, looking)

## Bindings

A **binding** connects a physical control to an action.

- **Bindings are data.** Defaults ship with the app; users override them (rebinding); both are layered like any
  other settings ([Settings](../04-settings.md)).
- **Composites** build one value from several controls: W/A/S/D become a 2D vector; two keys become one axis. This
  is the **Composite** pattern.
- **Processors** adjust the value: invert the Y axis, scale by sensitivity, apply a deadzone or curve.
- **Conflicts:** the rebinding screen detects two actions in the same context bound to the same control.

## Interactions

How a raw input becomes an action being "performed" (each one an interchangeable rule, the **Strategy** pattern):

- **press / release**
- **hold:** performed after being held for 0.5 s (canceled if released earlier)
- **tap:** pressed and released quickly
- **double tap**
- **chord / modifier:** Shift + click, Ctrl + wheel
- **threshold:** an analog trigger pressed past a point counts as a button

Actions report phases: **started** (the input began), **performed** (the interaction completed), **canceled**
(it began but didn't complete, e.g. a hold released too early). Charged attacks and "hold to confirm" buttons use
all three.

## Contexts (action maps)

A **context** is a set of actions active together: "Gameplay", "Menu", "Vehicle", "Editor camera".

- Contexts form a **stack**. Opening a menu **pushes** "Menu" on top of "Gameplay"; its bindings take priority and
  can block those below. Closing the menu **pops** it.
- The same key can mean different things in different contexts: Escape is "pause" in Gameplay and "back" in Menu,
  and neither has to know about the other.
- This is the **State** pattern as a stack, and it continues the routing chain
  ([Routing and focus](03-routing-and-focus.md)) inside the game.

## Virtual devices

An action is a **stand-in for whatever is bound to it**: an adapter between devices and intentions. So anything
that produces input can feed actions:

- **on-screen touch controls** (a virtual joystick, virtual buttons)
- **remote input:** a phone used as a controller, a network client's inputs replayed on the server
- **scripted input:** tests, bots, demo playback

To the application, a virtual device is indistinguishable from real hardware. This is what makes input testable
([Coupling](10-coupling.md)).
