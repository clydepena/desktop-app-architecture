# Using the UI toolkit

How the rest of an application builds and drives its UI.

## The doorways

| Doorway | What callers do with it | Typical calls |
|---|---|---|
| **Widgets** | build the UI | immediate: `button(label)`, `slider(label, ref v)`; retained: `new Button(...)`, `panel.add(child)`; declarative: `Column(Text(...), Button(...))` |
| **Layout** | arrange widgets | containers (`Row`, `Column`, `Grid`, `Dock`, `ScrollView`), sizing rules, virtualized lists |
| **Binding and events** | connect to data and actions | `bind(widget.value, viewModel.volume)`, `onClick(action)`, commands from the action registry |
| **Focus and navigation** | keyboard and gamepad | `focus(widget)`, tab order, directional navigation hints, focus scopes |
| **Styles and themes** | appearance | `applyTheme(dark)`, style sheets, design tokens, visual states |
| **Text** | labels and editing | localized text by key, text fields with input methods and clipboard |
| **Backends** | connect to the platform | a rendering backend (draw data -> a render pass), a platform backend (windows, clipboard, IME, accessibility) |
| **Input sharing** | cooperate with the input service | `wantsMouse()`, `wantsKeyboard()` each frame |
| **Accessibility** | expose the tree | roles, names, states per element; automatic for standard widgets |

## How each part of an application uses it

| Part | How it uses the UI toolkit |
|---|---|
| **Lifecycle** | creates the toolkit with its backends, feeds it input and frame time, asks it to render each frame (or when it requests a redraw) ([Lifecycle](../01-lifecycle.md)) |
| **Windowing** | the platform backend creates extra windows for detached panels, reads content scale and system theme, positions IME windows ([Windowing and display](../windowing/README.md)) |
| **Rendering** | the rendering backend draws the toolkit's draw data as one pass per window; viewport panels show render targets as images ([Rendering the UI](07-rendering-the-ui.md)) |
| **Input** | raw events go to the toolkit first; it reports what it consumed; the rest becomes game actions ([Input](../input/README.md)) |
| **Localization** | every label is a key; text goes through shaping, bidirectional ordering, line breaking and font fallback; layouts mirror for right to left ([Localization](../localization/README.md)) |
| **Time** | animations on real time; desktop redraws driven by need ([Time and scheduling](../time/README.md)) |
| **Tool framework** | menus from the action registry, inspectors from the data description, edits as undo commands, docking as UI state, the shared selection ([Tool UIs and product UIs](09-tool-and-product-uis.md)) |
| **Domain / gameplay** | exposes state and intentions to view models or presenters; never owned by the UI |
| **Audio** | UI sounds (clicks, notifications) as one-shot sound events on the UI bus ([Audio](../audio/README.md)) |
| **Settings** | theme, UI scale, font size, reduced motion, high contrast ([Settings](../04-settings.md)) |
| **Resources** | fonts, icons, images and styles loaded through handles, hot-reloaded in development ([Resources](../07-resources.md)) |
| **Scripting** | UI logic in scripts, through the same bindings and actions ([Scripting](../scripting/README.md)) |
