# The device layer and players

Turns what the OS reports into normalized, device-independent state, and keeps track of which devices exist and
who is using them.

## State and events

For every device, the layer keeps two views of the same input:

- **State:** what is true now. "A is down", "left stick at (0.4, -0.9)", "mouse at (812, 330)".
- **Events:** what happened, in order, with **timestamps**. "A pressed at 12.034 s", "A released at 12.101 s".

Both are needed: state for continuous things (a stick held at an angle), events for things that must not be missed
(a press and release between two reads).

## Normalization

- **Consistent ranges:** sticks from -1 to 1 on each axis, triggers from 0 to 1, buttons as 0 or 1 (or analog
  where the hardware supports it).
- **Deadzones:** a stick at rest rarely reads exactly zero. Values inside a small radius are treated as zero; the
  rest is rescaled so movement starts smoothly at the edge of the deadzone. A **radial** deadzone (on the stick's
  distance from center) feels better than one per axis.
- **Response curves:** small stick movements give fine control, large ones fast movement (a curve instead of a
  straight line).
- Settings for these come from **device profiles** ([Assets](11-assets.md)) and user settings.

## Devices come and go

- **Connected:** a new device appears. It gets an id, its type is identified (keyboard, a known gamepad model, an
  unknown pad mapped by the controller database), and it is offered to players.
- **Lost:** a gamepad is unplugged or its battery dies. An event is raised; games usually **pause** and ask for the
  controller back, rather than letting the player's character stand still and die.
- **Reconnected:** the same device (or a replacement) is given back to the same player.

## Players and device ownership

With local multiplayer, input must be split by player:

- Each player owns some devices: player 1 the keyboard and mouse, player 2 a gamepad.
- "Press A to join" assigns the pad that pressed A to a new player.
- Actions are read **per player** ([Action mapping](04-action-mapping.md)): "Jump" for player 2 only looks at
  player 2's devices.

Single-player apps usually treat every device as belonging to the one user, and switch the "current device type"
(for prompts) to whichever was used last.
