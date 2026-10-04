# Text input

Typing goes through a **separate path** from key events. Treating text as key presses works for one keyboard layout
in one language, and fails for most of the world.

## Characters, not keys

The OS delivers **character events**: the text actually produced, after:

- the **keyboard layout** (which character a key types)
- **modifiers** (Shift for capitals, AltGr for symbols on many European layouts)
- **dead keys:** on many layouts, ´ followed by e produces é; the first key alone produces nothing
- **input methods** (below)

One key press may produce no character, one character, or several; and several key presses may produce one.

## Input method editors (IME)

Chinese, Japanese, Korean and many other languages are typed through an **input method**: several keys form a
**composition** (shown underlined), the user picks the intended word from a **candidate list**, and only then is
the text committed.

The application must:

- start text input when a text field gains focus, and stop it when focus leaves
- tell the OS **where the caret is**, so the candidate window appears next to it
- show the **composition** in progress inside the field, separately from committed text
- accept the **committed** text when the user confirms

## Rules

- **Never build text from key-down events.** It breaks every layout but one and makes most languages impossible
  to type.
- **Key events still matter inside text fields** for editing keys (arrows, Backspace, Enter, Ctrl+A), which are not
  characters.
- **While a text field is active, game actions are suspended** ([Routing and focus](03-routing-and-focus.md)), so
  typing doesn't move the character or trigger shortcuts.
- **Paste, emoji and accessibility tools** insert text without any key presses at all; text fields must accept text
  from any source.
- **On touch devices**, starting text input shows the on-screen keyboard; the layout should leave the focused field
  visible above it.
