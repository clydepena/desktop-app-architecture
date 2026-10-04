# Coupling: what may depend on the UI toolkit

The direction rule: application UI code depends on the toolkit. The toolkit's **core** (widgets, layout, interaction,
styles) depends only on abstract **backends** and on runtime services below it (localization's text services, time,
resources). The **backends** are the adapters that know the renderer and the platform: the same arrangement as the
hardware interface under the renderer. Swap a backend and the toolkit runs somewhere else, including **headless** in
tests.

## Safe to couple

| System | Why it's safe |
|---|---|
| **Application UI -> toolkit** | using it is its purpose |
| **Toolkit core -> localization, time, resources** | runtime services below it |
| **Rendering backend -> renderer** | an adapter: draw data in, one render pass out |
| **Platform backend -> windowing, input, clipboard, accessibility APIs** | adapters to the platform |
| **Tool framework -> toolkit** | panels, inspectors and menus are built with it |

## Keep decoupled

| System | How it relates to the UI instead |
|---|---|
| **Domain / gameplay** | never depends on widgets; exposed through view models, presenters or bindings |
| **Documents and undo** | the UI never changes documents directly; it triggers actions and commands |
| **Renderer** | doesn't know UI internals; it receives draw data or shows a render target |
| **Input service** | doesn't know widgets; it only respects what the toolkit reports as consumed |
| **Application state** | lives outside widgets; widgets show it and request changes |

## Mistakes to avoid

- **Business logic inside UI callbacks:** untestable, and duplicated across screens.
- **The same state in a widget and in the application**, drifting apart.
- **Editor UI changing documents directly**, bypassing undo and dirty tracking.
- **Heavy work in UI event handlers**, freezing the whole UI; it belongs in jobs.
- **Fixed pixel sizes:** broken at other scales, languages and directions.
- **Duplicate ids in immediate mode** (two widgets sharing state), or rebuilding retained widget trees from scratch
  every frame.
- **No keyboard or gamepad navigation**, and no visible focus.
- **No accessibility tree** in a custom-drawn toolkit.
- **Redrawing continuously** in desktop applications.
- **Long lists and trees without virtualization.**
- **Animations on game time**, so menus freeze when the game pauses.

## Testing the boundary

- **Headless UI:** the toolkit with a null rendering backend and an injected-input platform backend; layout and
  interaction run in tests with no window and no GPU.
- **View model and presenter tests:** the UI's logic tested without any widgets, the main payoff of MVVM and MVP.
- **Layout tests:** sizes and positions at several scales, in pseudo-localization and in right to left.
- **Screenshot tests:** screens rendered offscreen and compared with references.
- **UI automation:** tests drive the application through the accessibility tree or test ids, the same tree screen readers
  use ([Accessibility](08-accessibility.md)).
