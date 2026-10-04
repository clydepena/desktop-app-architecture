# Interaction and focus

Getting each input event to exactly the right widget, and making every control reachable however the user interacts.

## Hit testing

Which widget is under the pointer? The **topmost** visible one whose shape contains the point, respecting **clipping**
(a widget scrolled out of its parent's view can't be clicked) and widgets marked as transparent to input (decorative
overlays).

## Event routing

An event goes to the widget under the pointer (or the one with focus, for keys), then **bubbles** up through its
parents until one handles it. Some toolkits also let parents see it first, on the way down ("tunneling"), so a dialog
can intercept Escape before a text field does.

This is the **Chain of Responsibility**, the same idea as [input routing](../input/03-routing-and-focus.md) and
[action context rules](../../tool-framework/05-actions.md): the most specific receiver first.

## Focus

- One widget has **keyboard focus**; it receives key events first.
- **Tab order** moves focus forward and back, in a sensible reading order (start to end, top to bottom).
- **Focus scopes** keep focus inside a dialog or a menu while it is open.
- Focus is **always visible**: a clear highlight, so keyboard and accessibility users know where they are.

## Capture

A drag that starts on a slider, a scroll bar or a splitter keeps receiving the mouse **even when the pointer leaves
it**, until the button is released. Without capture, fast drags drop as soon as the pointer slips off.

## Keyboard and gamepad navigation

Every control must be reachable without a mouse:

- **desktop:** Tab and Shift+Tab, arrow keys within lists and menus, Enter and Space to activate, Escape to close
- **game and TV UIs:** **directional navigation**: pressing right moves to the nearest control to the right, computed
  from widget positions, or set explicitly where the automatic choice is wrong

## Cooperating with the input service

Each frame, the toolkit reports whether it **wants** the mouse or the keyboard (the pointer is over a window, a text
field has focus). The input service then doesn't also turn that click or key press into a game action
([Routing and focus](../input/03-routing-and-focus.md)).

## Text editing

Text fields handle a lot on their own:

- caret movement and selection by **grapheme clusters**, words and lines
- clipboard: cut, copy, paste
- **input methods** with composition shown in place ([Text input](../input/06-text-input.md))
- a field's own small **undo history** while typing, separate from the document's history; the committed value becomes
  one document command

## Other interactions

- **drag and drop** within the UI and to and from other applications
  ([Clipboard and drag-and-drop](../../tool-framework/08-clipboard-and-drag-drop.md))
- **tooltips** after hovering for a moment, and on focus for keyboard users
- **context menus** from right-click, a menu key, or a gamepad button
- **gestures** on touch screens: tap, long press, swipe, pinch
