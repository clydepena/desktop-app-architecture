# Feedback and prompts

"Input" devices also take **output**, and the input service owns it, because it knows which devices exist and who
owns them.

## Haptics (rumble)

- **Patterns are data:** strength over time for each motor, like sound events for vibration ([Assets](11-assets.md)).
- Several patterns playing at once are **combined** (the strongest wins, or they are added up to a limit).
- Rumble is sent to the **right player's** device only.
- It **stops when the game pauses** or loses focus, and respects the user's "vibration off" setting.
- Phones and some controllers support finer haptics (short clicks, textures); patterns can describe those too.

## Lights and adaptive triggers

Controller light bars, keyboard lighting and adaptive trigger resistance are controlled the same way: as named
effects, per device, through the input service.

## Cursor control

- **Lock and hide** the cursor for camera control; restore it when a menu opens or focus is lost.
- **Confine** it to the window (useful for strategy games with edge scrolling).
- **Cursor shapes:** resize arrows, text beam, hand, busy.
- A visible **way out:** a locked cursor must always be releasable (Escape, losing focus), or the user is trapped.

## Button prompts

"Press **A**", "Press **✕**" and "Press **Space**" are the same prompt on different devices. The right one is chosen
from:

- the **action** the prompt is about
- the **current binding** of that action (so prompts stay correct after rebinding)
- the **device type** the player is using now, often switching to whichever device was used last

```
icon = input.glyphFor("Jump", player)   // keyboard icon, Xbox-style icon, PlayStation-style icon, touch icon
```

The icons come from **glyph sets** per device family ([Assets](11-assets.md)). Never write "Press A" into text
directly: refer to the action, and let the prompt system resolve it.
