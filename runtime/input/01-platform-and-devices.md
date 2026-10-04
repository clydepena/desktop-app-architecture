# Platform and devices

The lowest layer: what the operating system delivers, for each family of devices. The OS reports input as
**events** (window messages, callbacks) or through **polling APIs**, and both differ per platform. The platform
layer hides that behind one interface (the **Adapter / Bridge** pattern).

## Keyboard

Every key has two identities, and choosing the wrong one is a classic bug:

| Identity | What it means | Example | Use it for |
|---|---|---|---|
| **Scancode** | the physical position of the key | "the key where W is on a US keyboard" | movement and other layouts based on position (WASD) |
| **Keycode** | the meaning on the current layout | "the key that types Z" | shortcuts that refer to letters (Ctrl+Z), showing key names |

On a French AZERTY keyboard, the key in the W position types Z. Binding movement to scancodes keeps WASD in the
same physical place for everyone; binding Ctrl+Z to the keycode keeps undo on the key labeled Z.

Other keyboard details: modifier keys (Shift, Ctrl, Alt, Cmd/Super) on both sides, key repeat while held (useful
for text and menus, wrong for gameplay), and keys the OS reserves for itself.

## Mouse

- **Position** in window coordinates, for pointing at things.
- **Buttons** (often 5 or more) and the **wheel** (vertical and horizontal, sometimes in fine steps).
- **Raw motion:** unaccelerated movement deltas straight from the device, for camera control. OS pointer
  acceleration is good for desktop use and bad for aiming.
- **Cursor control:** hide it, lock it in place (for camera control), confine it to the window.

## Gamepads

- Buttons, two sticks, two analog triggers, a directional pad.
- **Many models, many layouts:** the OS reports raw buttons and axes; a **controller database** maps known models
  to a standard layout ("south button", "left stick X"), so the game sees the same thing for every pad.
- **Outputs:** rumble, lights, adaptive triggers ([Feedback and prompts](07-feedback-and-prompts.md)).
- **Wireless** pads connect and disconnect at any time, and report battery levels.

## Touch and pen

- **Several contact points** at once, each with an id that follows one finger from down to up.
- **Pressure, tilt and orientation** for pens; hovering above the screen.
- **Gestures** (tap, swipe, pinch, rotate) are recognized from contact points, either by the OS or by the input
  service.

## Sensors and others

Gyroscopes and accelerometers (motion aiming, tilting), joysticks, steering wheels, MIDI controllers, VR
controllers, eye trackers, adaptive controllers. Each is a device family with its own controls, which the device
layer normalizes ([2](02-device-layer-and-players.md)).

## Text input

Typing arrives through its **own path** (character and composition events), not through key events. It has its
own page: [Text input](06-text-input.md).
