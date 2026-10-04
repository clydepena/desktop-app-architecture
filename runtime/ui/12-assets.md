# Assets

Which assets a UI toolkit uses, and at which layer.

How source files are imported, cached and cooked into the runtime formats below is described in the
[asset pipeline](../../tool-framework/asset-pipeline/README.md).

## Assets by layer

| Layer | Assets it uses | What they hold |
|---|---|---|
| **Application UI** | **screens and layouts** | declarative layout files or UI prefabs built in a designer: widgets, containers, bindings |
| | **UI scripts** | screen logic, where it lives in scripts ([Scripting](../scripting/README.md)) |
| **Widgets** | **widget templates** | reusable composite controls (an inventory slot, a labeled slider) |
| **Style** | **themes and style sheets** | styles per widget type and visual state |
| | **design tokens** | named colors, spacing, radii, text styles |
| | **skins and nine-slice images** | decorative frames, buttons, panels |
| | **icon sets** | icon fonts, vector icons, or bitmaps at several scales |
| | **cursor sets** | through windowing ([Assets](../windowing/09-assets.md)) |
| **Text** | **fonts and text styles** | through localization's per-script font choices |
| **Animation** | **transitions and curves** | fades, slides, easing curves |
| **Sound** | **UI sound events** | through audio ([Sound events](../audio/06-sound-events.md)) |

## Strings aren't UI assets

Labels in screens are **keys**, resolved by [localization](../localization/README.md). A screen asset never contains
display text directly, so the same screen works in every language.

## Settings, UI state and runtime state

- **User settings, not assets:** theme, UI scale, font size, reduced motion, high contrast
  ([Settings](../04-settings.md)).
- **UI state, not assets:** docking layouts, window placement, open tree nodes, column widths
  ([UI state](../../tool-framework/09-ui-state.md)).
- **Runtime state:** the widget tree, focus, hover, animations in progress. Never saved.

## Hot reload

Screens, styles and themes reload while the application runs, through their asset handles, so designers see changes
immediately. Widget-local UI state (scroll positions, open nodes) is kept across reloads where the widgets still exist.
