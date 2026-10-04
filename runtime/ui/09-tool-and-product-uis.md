# Tool UIs and product UIs

The UI of an editor and the UI of the product it makes have different priorities.

| | Tool / editor UI | Product / game UI |
|---|---|---|
| **Priorities** | density, speed of building panels, keyboard and mouse | style, animation, gamepad and touch, reading from a distance (TVs) |
| **Typical model** | immediate mode, or a retained toolkit with many standard widgets | retained or declarative, built in a UI designer, heavily styled |
| **Special widgets** | docking, property grids, trees, tables, node graphs, viewports showing render targets | HUD elements, radial menus, health bars, minimaps, dialogue boxes |
| **Edits** | through actions and undo commands | through gameplay intentions |
| **Localization** | the editor's own strings | the product's content strings |
| **Who builds screens** | programmers, in code | UI designers and artists, in tools |

## Tool UIs

Built from the tool framework's parts:

- menus, toolbars and shortcuts from the **action registry** ([Actions](../../tool-framework/05-actions.md))
- inspectors generated from the **data description** ([Inspectors](../../tool-framework/07-inspectors.md))
- edits recorded as **undo commands** ([Undo](../../tool-framework/04-undo.md))
- docking layouts and panel state saved as **UI state** ([UI state](../../tool-framework/09-ui-state.md))
- the shared **selection** ([Selection and context](../../tool-framework/06-selection-and-context.md))

Immediate mode suits these well: panels are mostly views of data that changes constantly, written by programmers.

## Product UIs

- built visually by designers, saved as **screen assets** ([Assets](12-assets.md))
- connected to gameplay through **view models** or bindings ([State and data flow](03-state-and-data-flow.md))
- styled with skins, animated, localized, navigable with gamepads, readable at TV distance, scalable
- performance-sensitive, since they share the frame with the game

## Two toolkits

Many engines use **two toolkits**: an immediate-mode one for the editor and debug panels, and a retained or declarative
one for the product's UI. They share the same **rendering backend**, input routing and text services, so both draw
through the same renderer and receive input the same way. The product UI can also be previewed and edited inside the
editor, rendered into a viewport like the rest of the product.
