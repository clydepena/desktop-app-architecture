# UI toolkit

How an application turns its state into **interactive things on screen**: buttons, text fields, lists, menus, panels.

The UI toolkit is the biggest *consumer* of the other services: it lives in [windows](../windowing/README.md), draws
through [rendering](../rendering/README.md), receives [input](../input/README.md) first, shows
[localized](../localization/README.md) text, animates on [time](../time/README.md), and must be usable through
accessibility tools. It is used by both the **tools** (every editor panel) and the **shipped product** (menus, HUDs,
dialogs), so it sits at the top of the runtime layer, right under the application; it is the first runtime service
that is mostly about presentation rather than infrastructure.

Its special problems are **state and change**: keeping widgets in sync with the data they show without two copies
drifting apart; laying out under constant change (window sizes, scales, languages 40% longer, right to left); routing
every input event to exactly the right widget; and doing it fast enough to redraw every frame, or cheaply enough to use
almost no CPU when idle.

## The layers inside a UI toolkit

```
+--------------------------------------------------------------------+
| Application UI     editor panels, inspectors, game menus and HUD,    |  owned by the app
|                    dialogs                                            |
+--------------------------------------------------------------------+
| Widgets            buttons, text fields, lists, trees, tables,       |  reusable controls
|                    sliders, menus, tabs, docking, property grids      |
+--------------------------------------------------------------------+
| UI model           immediate, retained or declarative; state and     |  the architecture
|                    data binding                                       |
+--------------------------------------------------------------------+
| Layout             measure and arrange; stacks, grids, anchors,      |
|                    scrolling, virtualization; logical units; RTL      |
+--------------------------------------------------------------------+
| Interaction        hit testing, event routing, focus, capture,       |
|                    keyboard and gamepad navigation, drag and drop     |
+--------------------------------------------------------------------+
| Style and text     themes, visual states, animation; fonts and text  |
|                    layout (through localization's text services)     |
+--------------------------------------------------------------------+
| Backends           rendering backend (draw data -> a render pass),   |  adapters
|                    platform backend (windows, clipboard, IME, a11y)  |
+--------------------------------------------------------------------+
```

## Reading order

| # | Document | What it covers |
|---|---|---|
| 1 | [Native, custom-drawn or web](01-native-custom-or-web.md) | the three ways to put widgets on screen, and their trade-offs |
| 2 | [Immediate, retained and declarative](02-immediate-retained-declarative.md) | the central architectural choice |
| 3 | [State and data flow](03-state-and-data-flow.md) | one source of truth; MVC, MVP, MVVM, unidirectional flow; edits as commands |
| 4 | [Layout](04-layout.md) | measure and arrange, containers, sizing, units, localization, virtualization |
| 5 | [Interaction and focus](05-interaction-and-focus.md) | hit testing, routing, focus, capture, navigation, text editing |
| 6 | [Styling and theming](06-styling-and-theming.md) | styles, visual states, themes, tokens, skins, icons, animation |
| 7 | [Rendering the UI](07-rendering-the-ui.md) | draw data, batching, when to redraw, several windows, world-space UI |
| 8 | [Accessibility](08-accessibility.md) | the accessibility tree, and everything beyond screen readers |
| 9 | [Tool UIs and product UIs](09-tool-and-product-uis.md) | different priorities, and why engines often use two toolkits |
| 10 | [Using the UI toolkit](10-using-the-ui-toolkit.md) | the public interface, and how each part of an app uses it |
| 11 | [Coupling: what may depend on the UI toolkit](11-coupling.md) | core and backends; headless UI, view model tests, automation |
| 12 | [Assets](12-assets.md) | screens, templates, themes, skins, icons, fonts, transitions |

## How the UI toolkit compares

| | Windowing | Input | Localization | UI toolkit |
|---|---|---|---|---|
| Role | a place on screen | intentions in | text for every locale | **interactive views of the application's state** |
| Depends on | platform | platform | IO, resources, locale data | **backends to windowing, rendering and input; localization, time, resources** |
| Central problem | units and change | translation and routing | assumptions about text | **state sync, layout under change, routing to the right widget** |
| Abstraction | windows, surfaces | actions | keys | **widgets, layouts, bindings, draw data** |
| Test version | no window | injected input | pseudo-locales | **a headless toolkit with null backends** |

## Patterns in a UI toolkit

| Pattern | Where |
|---|---|
| Composite | the widget tree |
| Observer | events, observable properties, bindings |
| MVC / MVP / MVVM, unidirectional data flow | separating the UI from the application |
| Command | actions and undo commands triggered by the UI |
| Chain of Responsibility | events bubbling up through parents |
| Strategy | layout algorithms of containers |
| Decorator | wrappers adding borders, scrolling, tooltips |
| State | visual states (hovered, pressed, disabled) |
| Flyweight | styles shared by many widgets |
| Adapter | rendering and platform backends |
| Virtual proxy | virtualized rows created only when visible |
| Reconciliation | declarative UIs diffed against the retained tree |
| Memento | UI state saved and restored |

## Related

- [Routing and focus](../input/03-routing-and-focus.md): how input reaches the UI before the game.
- [Windowing and rendering](../windowing/06-windowing-and-rendering.md): where the UI's pixels end up.
- [Text rendering and layout](../localization/05-text-rendering-and-layout.md): what text in the UI must support.
- [Inspectors](../../tool-framework/07-inspectors.md), [Actions](../../tool-framework/05-actions.md),
  [UI state](../../tool-framework/09-ui-state.md): tool-framework parts built with the toolkit.
