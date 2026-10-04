# Routing and focus

One key press must reach **exactly one** place: the text field being typed in, the open menu, the focused panel,
or the game. Routing decides which.

## The chain

Events go through receivers from the most specific outwards (the **Chain of Responsibility** pattern):

```
1. a modal dialog is open?           -> it gets everything
2. a text field is being edited?     -> it gets the keys (typing)
3. the UI wants the mouse/keyboard?  -> the focused panel or widget gets it
4. otherwise                         -> the viewport / game gets it
```

The first receiver that handles an event **consumes** it, and it goes no further.

## Consumption

Without consumption, one event triggers several things: typing "w" in a chat box also walks the character forward;
pressing Escape closes the menu *and* opens the pause screen. Each receiver marks what it handled, and the next
ones in the chain never see it.

UI toolkits usually report whether they **want** the mouse or keyboard this frame (because the pointer is over a
window, or a text field has focus). The game layer checks that before reading input.

## Capture

A drag that starts inside a panel (a slider, a scrollbar, a viewport being orbited) keeps receiving mouse events
**even when the pointer leaves it**, until the button is released. Without capture, a fast drag "drops" as soon as
the pointer slips outside the widget.

## Focus

- **Window focus:** when the application loses focus (alt-tab, a notification, another window), every held key and
  button must be **released** in the input state. Otherwise the key that was down when focus left stays "down"
  forever: the stuck-key bug.
- **Panel focus:** inside an application, only the focused panel receives keyboard input. In an editor this is what
  separates editor shortcuts from a game running in a viewport
  ([Selection and context](../../tool-framework/06-selection-and-context.md)).
- **Pointer focus:** mouse events go to whatever is under the pointer, unless something has captured the mouse.

## Contexts are part of routing

Inside the game layer, routing continues through **contexts**: a menu on top of gameplay takes Escape, and gameplay
never sees it ([Action mapping](04-action-mapping.md)).
