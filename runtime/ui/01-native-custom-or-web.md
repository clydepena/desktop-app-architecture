# Native, custom-drawn or web

Three ways to put widgets on screen.

| Approach | What it is | Good | Bad |
|---|---|---|---|
| **Native widgets** | the operating system's own buttons, text fields, menus | look and behave native; accessibility, text input and IME come for free | different on every platform; hard to style; awkward inside a GPU viewport |
| **Custom-drawn** | the toolkit draws everything itself, through the renderer | identical everywhere; fully stylable; lives inside GPU views; the usual choice for games and creative tools | must rebuild text editing, IME support, accessibility and the native feel itself |
| **Embedded web view** | HTML and CSS rendered by an embedded browser engine | a huge ecosystem; designers already know it | heavy; a second runtime; harder to combine with GPU content |

## Where each fits

- **Business and productivity apps** often use native widgets or a web stack: their UI *is* the product, and native
  behavior and accessibility matter most.
- **Games** almost always draw their own UI: it must match the game's style, animate, work with gamepads and sit on top
  of (or inside) the rendered scene.
- **Engine editors and creative tools** (image, 3D, audio editors) usually draw their own UI too, because most of the
  window is GPU content (viewports, canvases) and the UI must sit right next to it.

## What custom-drawn toolkits must rebuild

Choosing custom-drawn means taking on what the OS would otherwise provide:

- **text editing**: caret, selection, clipboard, undo inside a field, input methods ([Text input](../input/06-text-input.md))
- **text rendering**: shaping, bidirectional text, line breaking, font fallback
  ([Text rendering and layout](../localization/05-text-rendering-and-layout.md))
- **accessibility**: an explicit accessibility tree ([Accessibility](08-accessibility.md))
- **platform conventions**: scrolling feel, keyboard shortcuts, menu behavior, file dialogs (often still native)

None of these is optional for a toolkit used by many people; planning for them up front is much cheaper than adding
them later.

## Mixing

Mixing is common: a custom-drawn application still uses **native file dialogs**, the system clipboard and OS
notifications ([OS integration](../windowing/05-os-integration.md)); a native application may embed a custom-drawn
canvas; a game may use a web view for a store page.
