# Coupling: what may depend on input

Input flows **up** as data, but dependencies still point **down**: consumers depend on the input service's
interface, and the input service depends only on the platform layer and the settings it reads. It never calls
gameplay, the UI or the editor; they pull actions or subscribe to action events.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Lifecycle** | pumps events and builds snapshots: the single driver |
| **Platform input APIs** | below the service |
| **UI toolkit** | first in the routing chain; designed to receive raw input |
| **Player controllers, cameras, editor tools** | read actions through the interface |
| **Settings** | store bindings and preferences, which the service reads |
| **Editor shortcuts, input debugger, recorder** | the tool layer, above input |

## Keep decoupled

| System | How it gets what it needs instead |
|---|---|
| **Gameplay / domain** | reads actions, never keys, buttons or devices |
| **Rendering and audio** | never read input; they present state that gameplay changed. The cursor is drawn by the UI; rumble is the input service's own output |
| **Networking** | carries action snapshots produced by the input layer; never reads devices |
| **Undo and documents** | record the *commands* that input triggered, never input events |
| **Physics** | receives forces and movement from gameplay, never input |

## Mistakes to avoid

- **Hard-coded keys in gameplay** (`if (key == SPACE)`): rebinding and other devices become impossible.
- **Checking keys in many places** instead of through actions and contexts.
- **Keycodes for movement** (breaks non-QWERTY layouts) or **scancodes for shortcuts** (shows the wrong letter).
- **Text built from key-down events** ([Text input](06-text-input.md)).
- **Missed presses** between fixed ticks; **camera speed tied to frame rate** ([Timing](05-timing.md)).
- **Game input processed while a text field or menu has focus**: typing moves the character.
- **Held keys not released on focus loss**: stuck keys after alt-tab.
- **Ignoring disconnects:** the game keeps running with nobody able to control it.

## Testing the boundary

- **Injected or scripted input** (virtual devices, the **Null Object** idea applied to hardware): tests drive the
  application through actions or simulated devices, with no hardware attached.
- **Record and replay:** action snapshots recorded from a session reproduce it exactly, if the simulation is
  deterministic.
- **Bots and AI players** use the same actions as people. If they can play, gameplay doesn't depend on devices.
