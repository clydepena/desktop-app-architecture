# Accessibility

A UI that can only be used with a mouse, by someone with perfect vision and hearing, excludes many people. Much of
accessibility lives in the UI toolkit.

## The accessibility tree

Screen readers and other assistive tools read the UI through the operating system's accessibility APIs (UI Automation
on Windows, NSAccessibility on macOS, AT-SPI on Linux). The toolkit exposes an **accessibility tree**, where every
element has:

- a **role:** button, checkbox, slider, list item, text field, dialog
- a **name:** "Save", "Music volume" (localized)
- a **state:** checked, disabled, expanded, selected, its value
- **actions:** activate, increment, expand

**Native widgets provide this for free. Custom-drawn toolkits must build it explicitly**, and it is the most commonly
skipped feature. Retained and declarative trees map onto it naturally; pure immediate-mode toolkits need an extra layer
that remembers the elements described each frame.

## Beyond screen readers

- **Full keyboard (and gamepad) operation,** with visible focus ([Interaction and focus](05-interaction-and-focus.md)).
- **Text and UI scaling** without layouts breaking ([Layout](04-layout.md)).
- **Contrast:** text and important elements readable against their background; a high-contrast theme
  ([Styling and theming](06-styling-and-theming.md)).
- **Never color alone:** errors, states and teams also shown by icons, text or patterns, for color-blind users.
- **Captions and subtitles** for audio, and visual cues for important sounds.
- **Reduced motion:** animations reduced or removed on request.
- **Timing:** nothing that must be done within a short time limit without a way to extend or disable it.
- **Input accessibility** (rebinding, hold vs toggle) is covered in [Accessibility in input](../input/08-accessibility.md).

## Testing it

- Navigate every screen with the keyboard only, then with a gamepad only.
- Use the platform's screen reader on every screen at least once per release.
- Check contrast with a tool, and the UI at 200% text scale.
- **UI automation** drives the application through the same accessibility tree
  ([Coupling](11-coupling.md)), so building it pays off twice.
